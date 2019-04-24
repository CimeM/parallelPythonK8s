# A guide to work queues with kubernetes

This tutorial will show you how to create a work queue with kubernetes. The core idea is to deploy a database as a list of tasks and make python workers do a task for every item on the list.

## Instructions

### 1. Build the worker container on your local machine:
`
docker build -t job-wq-2 .

`

Then, tag it

`
docker tag job-wq-2 <username>/job-wq-2
`
Ofc, you want to use your docker hub username here. In case you want to use another registry, change accordingly.


The last step is to push the container to the registry.
`
docker push <username>/job-wq-2
`

### 2. Establish a database pod on your node
Here we are just creating a Redis database and populating it with some *tasks*. These are just entries and could easily become input data for your jobs.

`
kubectl run -i -t temp --image redis --labels="job-wq-2" --command "/bin/sh"

`
This command will start a pod and enter it. Upon which you can insert new data:


`
redis-cli -h redis
rpush job2 "apple" "banana" "cherry" "lemon"
`
Now list the information you entered
`
lrange job2 0 -1
`
The list will look something like this:

1) "apple"

2) "banana"

3) "cherry"

4) "lemon"


### 3. Start your job!
`
kubectl apply -f job.yaml
`

his command will start the job and finish when containers come to a successful stop.


You can check out job status with.
`
kubectl describe jobs/job-wq-2
`
You will get names of certain pods, use it in the next command.

Next command will show you logs, where you can either troubleshoot or inspect progress during the job.
`
kubectl logs pods/<insert pod name>
`

If all went well, you will get information about successful termination. The job will not try to run until you tell it to run again.

You can clean all with:
`
kubectl delete rs,svc,job -l chapter=job-wk-2
`

Keep in mind that we tagged all necessary objects with the tag. If you have any other objects tagged with the same tag, this last command will terminate it. Be careful.
