## Rolling updates and Rollbacks

- **Understanding Rollouts and Versioning in a Deployment**:
  - When you create a deployment, it triggers a rollout, which creates a new deployment revision.
  - A deployment revision helps track the changes made to the deployment and enables you to roll back to a previous version if needed.

- **Two Types of Deployment Strategies**:
  - Recreate Strategy: This strategy involves destroying all existing instances and then creating new instances of the updated application version. It can result in downtime during the transition.
  - Rolling Update Strategy (Default): This strategy involves taking down the older version and bringing up the newer version one by one, ensuring a seamless upgrade without downtime.

- **Updating a Deployment**:
  - You can update a deployment by modifying its definition file and running the `kubectl apply` command or by using the `kubectl set image` command to update the image of your application.
  - When you update a deployment, a new rollout is triggered, and a new revision of the deployment is created.

- **Viewing Deployment Details**:
  - You can use the `kubectl describe deployment` command to see detailed information about the deployments, including the events and scaling actions.

- **Performing an Upgrade Under the Hood**:
  - When a new deployment is created, a replica set is automatically created, which then creates the required pods.
  - During an upgrade, the deployment creates a new replica set and starts deploying the containers there while scaling down the pods in the old replica set, following the rolling update strategy.

- **Rolling Back a Deployment**:
  - If you need to roll back a deployment, you can use the `kubectl rollout undo` command followed by the name of the deployment.
  - This will destroy the pods in the new replica set and bring back the older pods in the old replica set, reverting your application to its previous version.

- **Summary of Useful Commands**:
  - `kubectl create` - Create a deployment
  - `kubectl get deployments` - List the deployments
  - `kubectl apply` and `kubectl set image` - Update the deployments
  - `kubectl rollout status` - See the status of rollouts
  - `kubectl rollout history` - Check the revisions and history of a deployment
  - `kubectl describe deployment` - View detailed information about the deployments
  - `kubectl rollout undo` - Roll back a deployment operation.
