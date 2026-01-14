# Kubernetes Controllers and Operators

## Core Principles

1. **Reconciliation is idempotent** - Same input always produces same result
2. **One controller per CRD** - Single responsibility principle
3. **Level-triggered, not edge-triggered** - React to current state, not events
4. **Use status conditions** - Communicate resource state clearly
5. **Follow Kubernetes API conventions** - Consistency with ecosystem

---

## Controller Reconciliation Loop

**The reconciliation pattern:**

```go
// Reconciler reconciles a MyResource object
type Reconciler struct {
    client.Client
    Scheme *runtime.Scheme
}

// Reconcile is the main reconciliation loop
func (r *Reconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := log.FromContext(ctx)

    // 1. Fetch the resource
    var resource myapi.MyResource
    if err := r.Get(ctx, req.NamespacedName, &resource); err != nil {
        if apierrors.IsNotFound(err) {
            // Resource deleted, nothing to do
            return ctrl.Result{}, nil
        }
        // Error reading the object
        return ctrl.Result{}, err
    }

    // 2. Handle deletion (if using finalizers)
    if !resource.ObjectMeta.DeletionTimestamp.IsZero() {
        return r.reconcileDelete(ctx, &resource)
    }

    // 3. Add finalizer if needed
    if !controllerutil.ContainsFinalizer(&resource, myFinalizer) {
        controllerutil.AddFinalizer(&resource, myFinalizer)
        if err := r.Update(ctx, &resource); err != nil {
            return ctrl.Result{}, err
        }
        return ctrl.Result{}, nil
    }

    // 4. Reconcile the resource to desired state
    if err := r.reconcileNormal(ctx, &resource); err != nil {
        log.Error(err, "failed to reconcile resource")
        return ctrl.Result{}, err
    }

    return ctrl.Result{}, nil
}
```

---

## Idempotent Reconciliation

**Key principle: Same input → Same output**

```go
func (r *Reconciler) reconcileNormal(ctx context.Context, resource *myapi.MyResource) error {
    // Get desired state from spec
    desiredDeployment := r.buildDeployment(resource)

    // Get current state
    var currentDeployment appsv1.Deployment
    err := r.Get(ctx, client.ObjectKeyFromObject(desiredDeployment), &currentDeployment)

    if apierrors.IsNotFound(err) {
        // Resource doesn't exist, create it
        if err := r.Create(ctx, desiredDeployment); err != nil {
            return fmt.Errorf("failed to create deployment: %w", err)
        }
        return nil
    }

    if err != nil {
        return fmt.Errorf("failed to get deployment: %w", err)
    }

    // Resource exists, check if update needed
    if !reflect.DeepEqual(currentDeployment.Spec, desiredDeployment.Spec) {
        currentDeployment.Spec = desiredDeployment.Spec
        if err := r.Update(ctx, &currentDeployment); err != nil {
            return fmt.Errorf("failed to update deployment: %w", err)
        }
    }

    // Always return nil on success - idempotent!
    return nil
}
```

---

## Using Status Conditions

**Communicate resource state clearly:**

```go
// Define condition types
const (
    ConditionReady     = "Ready"
    ConditionDegraded  = "Degraded"
    ConditionAvailable = "Available"
)

// Update status with conditions
func (r *Reconciler) updateStatus(ctx context.Context, resource *myapi.MyResource, condition metav1.Condition) error {
    // Find existing condition
    existingCondition := meta.FindStatusCondition(resource.Status.Conditions, condition.Type)

    if existingCondition == nil || existingCondition.Status != condition.Status ||
        existingCondition.Reason != condition.Reason ||
        existingCondition.Message != condition.Message {

        // Update condition
        meta.SetStatusCondition(&resource.Status.Conditions, condition)

        // Update status subresource
        if err := r.Status().Update(ctx, resource); err != nil {
            return fmt.Errorf("failed to update status: %w", err)
        }
    }

    return nil
}

// Usage in reconciliation
func (r *Reconciler) reconcileNormal(ctx context.Context, resource *myapi.MyResource) error {
    // Try to reconcile
    if err := r.reconcileDeployment(ctx, resource); err != nil {
        // Mark as degraded
        condition := metav1.Condition{
            Type:    ConditionReady,
            Status:  metav1.ConditionFalse,
            Reason:  "ReconciliationFailed",
            Message: err.Error(),
        }
        r.updateStatus(ctx, resource, condition)
        return err
    }

    // Mark as ready
    condition := metav1.Condition{
        Type:    ConditionReady,
        Status:  metav1.ConditionTrue,
        Reason:  "ReconciliationSucceeded",
        Message: "Resource is ready",
    }
    return r.updateStatus(ctx, resource, condition)
}
```

---

## Finalizers for Cleanup

**Clean up external resources:**

```go
const myFinalizer = "myresource.example.com/finalizer"

func (r *Reconciler) reconcileDelete(ctx context.Context, resource *myapi.MyResource) (ctrl.Result, error) {
    if controllerutil.ContainsFinalizer(resource, myFinalizer) {
        // Perform cleanup
        if err := r.cleanupExternalResources(ctx, resource); err != nil {
            // Cleanup failed, retry
            return ctrl.Result{}, err
        }

        // Remove finalizer
        controllerutil.RemoveFinalizer(resource, myFinalizer)
        if err := r.Update(ctx, resource); err != nil {
            return ctrl.Result{}, err
        }
    }

    return ctrl.Result{}, nil
}

func (r *Reconciler) cleanupExternalResources(ctx context.Context, resource *myapi.MyResource) error {
    // Delete external resources (cloud resources, webhooks, etc.)
    // This must be idempotent!

    log := log.FromContext(ctx)
    log.Info("cleaning up external resources", "name", resource.Name)

    // Example: Delete cloud storage bucket
    if resource.Status.BucketName != "" {
        if err := deleteCloudBucket(resource.Status.BucketName); err != nil {
            return fmt.Errorf("failed to delete bucket: %w", err)
        }
    }

    return nil
}
```

---

## Owner References

**Establish ownership for garbage collection:**

```go
func (r *Reconciler) reconcileDeployment(ctx context.Context, resource *myapi.MyResource) error {
    deployment := &appsv1.Deployment{
        ObjectMeta: metav1.ObjectMeta{
            Name:      resource.Name + "-deployment",
            Namespace: resource.Namespace,
        },
        Spec: appsv1.DeploymentSpec{
            // ... deployment spec
        },
    }

    // Set owner reference - deployment will be deleted when resource is deleted
    if err := controllerutil.SetControllerReference(resource, deployment, r.Scheme); err != nil {
        return fmt.Errorf("failed to set owner reference: %w", err)
    }

    // Create or update
    if err := r.Create(ctx, deployment); err != nil {
        if apierrors.IsAlreadyExists(err) {
            return r.Update(ctx, deployment)
        }
        return err
    }

    return nil
}
```

---

## Requeue Strategies

**Control when reconciliation runs again:**

```go
func (r *Reconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // ... fetch resource ...

    if err := r.reconcileNormal(ctx, &resource); err != nil {
        // Temporary error - requeue with backoff
        return ctrl.Result{}, err
    }

    // Check if resource needs periodic reconciliation
    if resource.Spec.RequiresPolling {
        // Requeue after specific duration
        return ctrl.Result{RequeueAfter: 5 * time.Minute}, nil
    }

    // Success - don't requeue (will reconcile on changes)
    return ctrl.Result{}, nil
}

// For rate-limited APIs
func (r *Reconciler) handleRateLimitError(err error) (ctrl.Result, error) {
    if isRateLimitError(err) {
        // Requeue after delay
        return ctrl.Result{RequeueAfter: 30 * time.Second}, nil
    }
    return ctrl.Result{}, err
}
```

---

## Watching Related Resources

**React to changes in dependent resources:**

```go
func (r *Reconciler) SetupWithManager(mgr ctrl.Manager) error {
    return ctrl.NewControllerManagedBy(mgr).
        For(&myapi.MyResource{}).
        Owns(&appsv1.Deployment{}). // Watch Deployments owned by MyResource
        Owns(&corev1.Service{}).    // Watch Services owned by MyResource
        Watches(
            &source.Kind{Type: &corev1.ConfigMap{}},
            handler.EnqueueRequestsFromMapFunc(r.findResourcesForConfigMap),
        ).
        Complete(r)
}

// Map ConfigMap changes to MyResource reconciliation
func (r *Reconciler) findResourcesForConfigMap(obj client.Object) []reconcile.Request {
    configMap := obj.(*corev1.ConfigMap)

    // Find all MyResources that reference this ConfigMap
    var resources myapi.MyResourceList
    if err := r.List(context.Background(), &resources); err != nil {
        return []reconcile.Request{}
    }

    var requests []reconcile.Request
    for _, resource := range resources.Items {
        if resource.Spec.ConfigMapName == configMap.Name {
            requests = append(requests, reconcile.Request{
                NamespacedName: types.NamespacedName{
                    Name:      resource.Name,
                    Namespace: resource.Namespace,
                },
            })
        }
    }

    return requests
}
```

---

## Validation Webhooks

**Validate resources before admission:**

```go
// Implement admission.Validator interface
func (r *MyResource) ValidateCreate() error {
    return r.validate()
}

func (r *MyResource) ValidateUpdate(old runtime.Object) error {
    return r.validate()
}

func (r *MyResource) ValidateDelete() error {
    return nil // Usually no validation needed for delete
}

func (r *MyResource) validate() error {
    var allErrs field.ErrorList

    // Validate spec fields
    if r.Spec.Replicas < 1 {
        allErrs = append(allErrs, field.Invalid(
            field.NewPath("spec", "replicas"),
            r.Spec.Replicas,
            "must be at least 1",
        ))
    }

    if r.Spec.Image == "" {
        allErrs = append(allErrs, field.Required(
            field.NewPath("spec", "image"),
            "image is required",
        ))
    }

    if len(allErrs) == 0 {
        return nil
    }

    return apierrors.NewInvalid(
        schema.GroupKind{Group: "myapi.example.com", Kind: "MyResource"},
        r.Name,
        allErrs,
    )
}
```

---

## Default Values with Webhooks

**Set defaults for optional fields:**

```go
// Implement admission.Defaulter interface
func (r *MyResource) Default() {
    if r.Spec.Replicas == 0 {
        r.Spec.Replicas = 1
    }

    if r.Spec.Port == 0 {
        r.Spec.Port = 8080
    }

    if r.Spec.ServiceType == "" {
        r.Spec.ServiceType = corev1.ServiceTypeClusterIP
    }
}
```

---

## Testing Controllers

**Test reconciliation logic:**

```go
func TestReconciler_Reconcile(t *testing.T) {
    scheme := runtime.NewScheme()
    _ = myapi.AddToScheme(scheme)
    _ = corev1.AddToScheme(scheme)
    _ = appsv1.AddToScheme(scheme)

    tests := []struct {
        name           string
        existingObjs   []client.Object
        resource       *myapi.MyResource
        wantErr        bool
        wantCondition  metav1.ConditionStatus
    }{
        {
            name: "creates deployment when missing",
            resource: &myapi.MyResource{
                ObjectMeta: metav1.ObjectMeta{
                    Name:      "test-resource",
                    Namespace: "default",
                },
                Spec: myapi.MyResourceSpec{
                    Replicas: 3,
                    Image:    "nginx:latest",
                },
            },
            wantErr:       false,
            wantCondition: metav1.ConditionTrue,
        },
        {
            name: "updates existing deployment",
            existingObjs: []client.Object{
                &appsv1.Deployment{
                    ObjectMeta: metav1.ObjectMeta{
                        Name:      "test-resource-deployment",
                        Namespace: "default",
                    },
                    Spec: appsv1.DeploymentSpec{
                        Replicas: ptr.To(int32(1)), // Old value
                    },
                },
            },
            resource: &myapi.MyResource{
                ObjectMeta: metav1.ObjectMeta{
                    Name:      "test-resource",
                    Namespace: "default",
                },
                Spec: myapi.MyResourceSpec{
                    Replicas: 5, // New value
                    Image:    "nginx:latest",
                },
            },
            wantErr:       false,
            wantCondition: metav1.ConditionTrue,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Create fake client with existing objects
            objs := append(tt.existingObjs, tt.resource)
            fakeClient := fake.NewClientBuilder().
                WithScheme(scheme).
                WithObjects(objs...).
                WithStatusSubresource(tt.resource).
                Build()

            r := &Reconciler{
                Client: fakeClient,
                Scheme: scheme,
            }

            // Run reconciliation
            req := reconcile.Request{
                NamespacedName: types.NamespacedName{
                    Name:      tt.resource.Name,
                    Namespace: tt.resource.Namespace,
                },
            }

            _, err := r.Reconcile(context.Background(), req)

            if (err != nil) != tt.wantErr {
                t.Errorf("Reconcile() error = %v, wantErr %v", err, tt.wantErr)
                return
            }

            // Verify status condition
            var updated myapi.MyResource
            if err := fakeClient.Get(context.Background(), req.NamespacedName, &updated); err != nil {
                t.Fatalf("failed to get updated resource: %v", err)
            }

            condition := meta.FindStatusCondition(updated.Status.Conditions, ConditionReady)
            if condition == nil {
                t.Error("expected Ready condition to be set")
            } else if condition.Status != tt.wantCondition {
                t.Errorf("got condition status %v, want %v", condition.Status, tt.wantCondition)
            }
        })
    }
}
```

---

## Best Practices Summary

1. **Idempotent reconciliation** - Same input always produces same result
2. **One controller per CRD** - Single responsibility
3. **Use status conditions** - Clear state communication
4. **Owner references** - Automatic garbage collection
5. **Finalizers for cleanup** - Handle external resource deletion
6. **Level-triggered** - React to current state, not events
7. **Error handling** - Return errors for retry with backoff
8. **Requeue strategically** - Only when necessary
9. **Validate with webhooks** - Prevent invalid resources
10. **Test reconciliation** - Use fake clients
11. **Follow API conventions** - Consistency with Kubernetes ecosystem
12. **Log appropriately** - Use structured logging

---

## References

- [Kubebuilder Book](https://book.kubebuilder.io/)
- [Kubebuilder Best Practices](https://kubebuilder.io/reference/good-practices)
- [Kubernetes API Conventions](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md)
- [Controller Runtime](https://github.com/kubernetes-sigs/controller-runtime)
