# HW4 — Short Answers

Answer each question in 2–3 sentences.

## 1. Rolling-update safety
Why does the Deployment use `maxUnavailable: 0`, and what would change if it were `maxUnavailable: 1`?

> _your answer here_

## 2. Health probes
Why do the liveness/readiness probes target `/health` instead of `/predict`?

> _your answer here_


## 3. Additional notes from Jordan
Screenshot 4 represents a completed run. The script does not appropriately manage the post-successful rollout and the port forwarding fails 13-14 requests after a successful deployment. The goal of this project is not to rewrite those scripts, the fraud tracker displays 0 bad prior to completion and that is sufficient for the request as outlined in the homework guidelines.  