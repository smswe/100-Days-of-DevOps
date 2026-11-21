# Day 16 - Configure Nginx as a Load Balancer

## Objective

Configure Nginx on the Load Balancer (LBR) server to distribute incoming HTTP requests across all three Apache application servers in the Nautilus infrastructure.

## Original Task

Day by day traffic is increasing on one of the websites managed by the `Nautilus` production support team. Therefore, the team has observed a degradation in website performance. Following discussions about this issue, the team has decided to deploy this application on a high availability stack i.e on `Nautilus` infra in `Stratos DC`. They started the migration last month and it is almost done, as only the LBR server configuration is pending.

Configure LBR server as per the information given below:

1. Install `nginx` on the `LBR` (load balancer) server if it is not already installed.
2. Configure load-balancing with the `http` context making use of all `App Servers`. Ensure that you update only the main Nginx configuration file located at `/etc/nginx/nginx.conf`.
3. Make sure you do not update the Apache port that is already defined in the Apache configuration on all app servers. Also make sure Apache service is up and running on all the app servers.
4. Once done, access the website by running `curl http://stlb01:80` in the terminal.

---

## 1. Infrastructure Details

| Server | Role | Service |
|---|---|---|
| stlb01 | Load Balancer | Ngin
