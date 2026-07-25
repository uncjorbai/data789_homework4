# HW4 — Short Answers

Answer each question in 2–3 sentences.

## 1. Rolling-update safety
Why does the Deployment use `maxUnavailable: 0`, and what would change if it were `maxUnavailable: 1`?

> This is where we get zero-downtime. maxUnavailable: 1 means that a single pod, of the three, is allowed to go down before being replaced. This can make deployment faster but also means that traffic is limited by 33%.

## 2. Health probes
Why do the liveness/readiness probes target `/health` instead of `/predict`?

> one of the responses is in the manifest. /health returns 200 WITHOUT touching Redis, so pods report Ready before Redis is up. Health just checks that the pods are available and ready for traffic. there's no predictions being run, just that the run can be predicted on. 

## 3. HPA:
Your HPA scales at 40% CPU up to 8 pods — if request volume doubled, what would you expect to happen, and what happens once it reaches the maximum?

> doubling would bring HPA above its threshold. The HPA adds pods to bring the average back down. It is capped at 8 pods, so the 8 pods compute everything and will eventually hit the limit. Once htat's done there is likely downtime, or a throttle in the traffic that can be moved through the system. 

## Note. Additional notes from Jordan
Screenshot 4 represents a completed run. The script does not appropriately manage the post-successful rollout and the port forwarding fails 13-14 requests after a successful deployment. The goal of this project is not to rewrite those scripts, the fraud tracker displays 0 bad prior to completion and that is sufficient for the request as outlined in the homework guidelines.  

## Note. 4 regions did not run before we reached the centralus. You should update the documentation to explicitly direct there. The list provided is inaccurate.
