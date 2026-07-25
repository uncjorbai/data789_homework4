DATA 789 - HW4 - Model Deployment & Scaling

Production-ready deployment of TrustBank's fraud-detection service (the provided Project 2 image) on Kubernetes, with autoscaling, zero-downtime updates, and a cloud deployment to Azure Container Apps.

The application image is pre-built and pulled as-is (ghcr.io/rfsalas/trustbank-fraud:v1). This repo is the deployment layer.

REPOSITORY LAYOUT

k8s/ : core manifests (Deployment, Service, HPA, Redis, ConfigMap)
k8s/blue-green/ : blue and green Deployments behind one Service
scripts/ : local bring-up, rolling-update demo, blue-green switch, smoke test
deploy_azure.sh : Azure Container Apps deploy (run in Azure Cloud Shell)
teardown_azure.sh : deletes the Azure resource group
ANSWERS.md : short-answer responses
screenshots/ : deployment evidence

PART 1 - DEPLOY & SCALE (KUBERNETES)

Local bring-up on Minikube:

bash scripts/local_up.sh

This starts Minikube, enables metrics-server, and applies Redis, the API Deployment, the Service, and the HPA.

Configuration applied per the assignment:

3 replicas, with CPU/memory requests and liveness, readiness, and startup probes
Right-sized limits: CPU 250m, memory 192Mi
owner: jorbai label on the Deployment and pod template
HPA tuned to 40% CPU utilization, min 3 and max 8 replicas
Service exposed as a LoadBalancer

Verify:

kubectl get pods,svc,hpa
kubectl port-forward svc/trustbank-fraud 8080:80 (in a second terminal)
bash scripts/smoke_test.sh http://localhost:8080 (POST /predict returns 200)

Self-healing is demonstrated by deleting a pod and watching the Deployment recreate it back to 3/3.

PART 2 - ZERO-DOWNTIME UPDATES

Rolling update (kubectl set image to v2, with a request loop counting drops):

bash scripts/rolling_update_demo.sh http://localhost:8080 ghcr.io/rfsalas/trustbank-fraud:v2

maxUnavailable: 0 plus maxSurge: 1 mean a new pod becomes Ready before any old pod is removed, so capacity never dips during the roll. See the note in ANSWERS.md about the trailing port-forward failures. The bad counter is 0 throughout the actual rollout.

Blue-green (two colors behind one Service, flipped via the selector):

kubectl apply -f k8s/blue-green/
bash scripts/bluegreen_switch.sh green (cut over)
bash scripts/bluegreen_switch.sh blue (roll back)

PART 3 - SHIP TO AZURE (CONTAINER APPS)

Run in Azure Cloud Shell (campus policy blocks Azure CLI sign-in on personal devices):

az account set --subscription "Azure for Students"
export USER=jorbai
bash deploy_azure.sh

The script imports the provided image into Azure Container Registry, deploys to Azure Container Apps with a user-assigned managed identity for the pull (scale-to-zero), and prints a public URL. Verify /health and /predict, then tear down the same day:

bash teardown_azure.sh

Region note: the kit default (eastus2) and westus3 were both blocked by my Azure for Students region policy (RequestDisallowedByAzure). The deploy succeeded on northcentralus. See ANSWERS.md.

NOTES ON FIXES

Two adjustments were needed for the pods to start:

REDIS_PORT collision: Kubernetes auto-injects a REDIS_PORT env var (set to tcp://<ip>:6379) for the redis Service, which the app tried to parse as an integer and crashed. Fixed by explicitly setting REDIS_PORT: "6379" in the Deployment env (and in both blue-green Deployments), so the app's value wins.
HPA metrics: metrics-server must be enabled for the HPA to report a real CPU target rather than <unknown> (handled by local_up.sh).

SCREENSHOTS

See screenshots/ for: pods 3/3 with Service and HPA to spec, a successful /predict, self-healing recovery, the rolling-update run, the blue-green switch, and the Azure deployment with teardown.
