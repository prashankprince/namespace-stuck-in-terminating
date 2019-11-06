# namespace-stuck-in-terminating
Manually delete your namespace that is stuck in a terminating state.
Steps below shows how you can manually delete terminating namespaces.


## Process involved -- 

1. Run below command to check which namespace is in Terminating state

    ````
    kubectl get ns
    ````
    
2. Run below command to see if finalizers field is present and store it in temporary json file

    ````
    kubectl get ns <terminating-namespace> -o json >tmp.json
    ````
    
    ````
    cat tmp.json
    ````
    will resemble like below:
    
    ````
    {
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "creationTimestamp": "2019-09-10T07:08:51Z",
        "deletionTimestamp": "2019-11-06T06:12:59Z",
        "labels": {
            "name": "<terminating-namespace>"
        },
        "name": "<terminating-namespace>",
        "resourceVersion": "7547753",
        "selfLink": "/api/v1/namespaces/<terminating-namespace>",
        "uid": "0c9cf468-95be-4faf-a989-7b71f8233a36"
    },
    "spec": {
        "finalizers": [
            "kubernetes"
        ]
    },
    "status": {
        "phase": "Terminating"
    }
    }
    ````
    
3. Remove finalizers field in tmp.json and it should resemble like below

    ````
    {
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "creationTimestamp": "2019-09-10T07:08:51Z",
        "deletionTimestamp": "2019-11-06T06:12:59Z",
        "labels": {
            "name": "<terminating-namespace>"
        },
        "name": "<terminating-namespace>",
        "resourceVersion": "7547753",
        "selfLink": "/api/v1/namespaces/<terminating-namespace>",
        "uid": "0c9cf468-95be-4faf-a989-7b71f8233a36"
    },
    "spec": {
        "finalizers": [
        ]
    },
    "status": {
        "phase": "Terminating"
    }
    }
    ````
    
4. To set a temporary proxy IP and port, run the following command. Be sure to keep your terminal window open until you delete the stuck namespace:

    ````
    kubectl proxy
    ````

5. From a new terminal window, make an API call with your temporary proxy IP and port like below:

    ````
    curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://127.0.0.1:8001/api/v1/namespaces/<terminating-            namespace>/finalize
    ````
    
6. Output of above must have finalizers removed as per below:

    ````
        {
      "kind": "Namespace",
      "apiVersion": "v1",
      "metadata": {
        "name": "<terminating-namespace>",
        "selfLink": "/api/v1/namespaces/<terminating-namespace>/finalize",
        "uid": "0c9cf468-95be-4faf-a989-7b71f8233a36",
        "resourceVersion": "7547753",
        "creationTimestamp": "2019-09-10T07:08:51Z",
        "deletionTimestamp": "2019-11-06T06:12:59Z",
        "labels": {
          "name": "<terminating-namespace>"
        }
      },
      "spec": {

      },
      "status": {
        "phase": "Terminating"
      }

    ````
    
7. Check if namespace is removed :

    ````
    kubectl get ns
    ```` 
