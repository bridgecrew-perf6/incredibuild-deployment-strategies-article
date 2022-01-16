
#  Incredibuild deployment strategies article tools

Contains all files and commands needed for Incredibuild deployment strategies article.


## **implementing recreate deployment:**
Deploy the first version of the application
```bash
kubectl apply -f version1-deployment.yml && kubectl rollout status -f version1-deployment.yml
```

Apply the production service

```bash
kubectl apply -f production-service.yml
```

Delete the first version of the application
```bash
kubectl delete -f version1-deployment.yml
```
  
After the first version is completely deleted, Deploy the new version of the application
```bash
kubectl apply -f version2-deployment.yml && kubectl rollout status -f version2-deployment.yml
```

Now, all that's left is to change the service to point to the new version.
```bash
kubectl patch service production-service --patch '{"spec": {"selector": {"version": "v1.1"}}}'
```


## **implementing Blue/Green deployment:**
Apply the blue instance’s deployment
```bash
kubectl apply -f version1-deployment.yml && kubectl rollout status -f version1-deployment.yml
```  

Apply the green instance’s deployment
```bash
kubectl apply -f version2-deployment.yml && kubectl rollout status -f version2-deployment.yml
```

Apply the production service
```bash
kubectl apply -f production-service.yml
```

**Run tests on the green instance**

If issues were discovered in the new version during testing, Delete the new version’s deployment and fix the application’s code before redeploying
```bash
kubectl delete -f version2-deployment.yml
```

If the new version testing was successful and the version is production-ready, change the service to point to the green instance
```bash
kubectl patch service production-service --patch '{"spec": {"selector": {"version": "v1.1"}}}'
```

and that’s it!


## **implementing Canary deployment:**

Deploy 3 instances of the first version of the application
```bash
kubectl apply -f version1-deployment.yml && kubectl rollout status -f version1-deployment.yml
```

Apply the production service
```bash
kubectl apply -f production-service.yml
```

Decrease the current version’s instances to 3
```bash
kubectl scale --replicas=3 -f version1-deployment.yml && kubectl rollout status -f version1-deployment.yml
```

Deploy the new version of the application on 3 instances
```bash
kubectl apply -f version2-deployment.yml && kubectl scale --replicas=3 -f version2-deployment.yml && kubectl rollout status -f version2-deployment.yml
```

Change the service to point to both versions, when 3 instances are the current version and 3 are the new one. In other words, 50% of the users will access the new version and 50% will access the current one
```bash
kubectl patch service production-service --patch '{"spec": {"selector": {"version": null}}}'
```

**Run tests on the new version’s instances**

If issues were discovered in the new version during testing, Rollback to the previous version (Delete the new version’s deployment, and increase the previous version’s instance number to the original)
```bash
kubectl delete -f version2-deployment.yml && kubectl scale --replicas=6 -f version1-deployment.yml && kubectl rollout status -f version1-deployment.yml
```

If the version is functioning well, complete the rollout by deleting the current version’s deployment and increasing the new version’s instance number to the original
```bash
kubectl delete -f version1-deployment.yml && kubectl scale --replicas=6 -f version2-deployment.yml && kubectl rollout status -f version2-deployment.yml
```
