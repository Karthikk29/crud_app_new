# How Your CI/CD Pipeline Works (Step-by-Step)

Here is exactly what happens when you run `git push`:

## Phase 1: GitHub Actions (The "CI" Part)
1.  **Trigger:** GitHub sees a new commit on the `main` branch.
2.  **Server Start:** GitHub starts a temporary Linux server (runner) in the cloud.
3.  **Checkout:** It downloads your code to this runner.
4.  **Build:** It runs `docker build` to create new images for your Frontend and Backend.
5.  **Push:** It logs in to **Docker Hub** and uploads these new images.
    *   *Result:* Your new code is now stored as a Docker Image on the internet.

## Phase 2: Deployment to EC2 (The "CD" Part)
6.  **Connect:** The GitHub runner uses your `EC2_SSH_KEY` to SSH into your EC2 instance (just like you do from your laptop).
7.  **Pull Code:** It runs `git pull` inside your EC2 folder to make sure it has the latest `docker-compose.prod.yml` file.
8.  **Pull Images:** It runs `docker-compose pull`. This downloads the **new** images from Docker Hub that were just built in Step 5.
9.  **Restart:** It runs `docker-compose down` (stops old containers) and `docker-compose up -d` (starts new ones).
10. **Cleanup:** It runs `docker image prune` to delete old, unused images to save space.

## Why it might fail?
*   **SSH Key:** If the key in GitHub Secrets is wrong, Step 6 fails.
*   **Permissions:** If the EC2 user can't run docker, Step 8 fails.
*   **Wrong File:** If it uses the wrong `docker-compose.yml` (like before), it builds locally instead of pulling the new images.

## Visual Flow
`You (Push)` -> `GitHub (Builds Image)` -> `Docker Hub (Stores Image)` -> `EC2 (Pulls Image & Restarts)`
