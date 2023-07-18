# Serverless RDS Proxy Demo

| Key          | Value                                                                |
| ------------ | -------------------------------------------------------------------- |
| Environment  | <img src="https://img.shields.io/badge/LocalStack-deploys-4D29B4.svg?logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAYAAACqaXHeAAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAKgAAACoABZrFArwAAABl0RVh0U29mdHdhcmUAd3d3Lmlua3NjYXBlLm9yZ5vuPBoAAALbSURBVHic7ZpNaxNRFIafczNTGIq0G2M7pXWRlRv3Lusf8AMFEQT3guDWhX9BcC/uFAr1B4igLgSF4EYDtsuQ3M5GYrTaj3Tmui2SpMnM3PlK3m1uzjnPw8xw50MoaNrttl+r1e4CNRv1jTG/+v3+c8dG8TSilHoAPLZVX0RYWlraUbYaJI2IuLZ7KKUWCisgq8wF5D1A3rF+EQyCYPHo6Ghh3BrP8wb1en3f9izDYlVAp9O5EkXRB8dxxl7QBoNBpLW+7fv+a5vzDIvVU0BELhpjJrmaK2NMw+YsIxunUaTZbLrdbveZ1vpmGvWyTOJToNlsuqurq1vAdWPMeSDzwzhJEh0Bp+FTmifzxBZQBXiIKaAq8BBDQJXgYUoBVYOHKQRUER4mFFBVeJhAQJXh4QwBVYeHMQJmAR5GCJgVeBgiYJbg4T8BswYPp+4GW63WwvLy8hZwLcd5TudvBj3+OFBIeA4PD596nvc1iiIrD21qtdr+ysrKR8cY42itCwUP0Gg0+sC27T5qb2/vMunB/0ipTmZxfN//orW+BCwmrGV6vd63BP9P2j9WxGbxbrd7B3g14fLfwFsROUlzBmNM33XdR6Meuxfp5eg54IYxJvXCx8fHL4F3w36blTdDI4/0WREwMnMBeQ+Qd+YC8h4g78wF5D1A3rEqwBiT6q4ubpRSI+ewuhP0PO/NwcHBExHJZZ8PICI/e73ep7z6zzNPwWP1djhuOp3OfRG5kLROFEXv19fXP49bU6TbYQDa7XZDRF6kUUtEtoFb49YUbh/gOM7YbwqnyG4URQ/PWlQ4ASllNwzDzY2NDX3WwioKmBgeqidgKnioloCp4aE6AmLBQzUExIaH8gtIBA/lFrCTFB7KK2AnDMOrSeGhnAJSg4fyCUgVHsolIHV4KI8AK/BQDgHW4KH4AqzCQwEfiIRheKKUAvjuuu7m2tpakPdMmcYYI1rre0EQ1LPo9w82qyNziMdZ3AAAAABJRU5ErkJggg=="> |
| Services     | API gateway, RDS Proxy, Lambda, Amazon Aurora                                    |
| Integrations | Serverless Framework, SAM, AWS SDK, Cloudformation     |
| Categories   | Serverless, Lambda Functions, Load Testing |
| Level        | Intermediate                                                         |
| Github       | [Repository link](https://github.com/localstack/sample-serverless-rds-proxy-demo) |     


# Introduction

This project demos benefits of using RDS proxy with serverless workload which depends on relational database like RDS Aurora.
Project shows end to end automated setup of RDS Aurora(PostgreSQL) with RDS proxy. Basic serverless architecture is set up 
using API gateway HTTP API and Lambda Functions.

Project sets up two endpoints with HTTP API, one which talks directly to RDS Aurora cluster and the other which talks 
via RDS Proxy. It provides load testing setup to measure the benefits of using RDS proxy in terms of connection pooling 
and elasticity.

This project assumes you already have RDS Aurora PostgreSQL cluster up and running. An RDS proxy instance
is also setup with force IAM authentication enabled. You can choose to create rds cluster with proxy following 
steps [below](#deploy-rds-aurora-cluster-with-rds-proxy) to have aurora cluster and 
RDS proxy setup.

## Architecture

The following diagram shows the architecture that this sample application builds and deploys:
![img.png](images/architecture.png)


## Prerequisites

* LocalStack Pro with the [`localstack` CLI](https://docs.localstack.cloud/getting-started/installation/#localstack-cli).
* [Serverless Application Model](https://docs.localstack.cloud/user-guide/integrations/aws-sam/) with the [samlocal](https://github.com/localstack/aws-sam-cli-local) installed.
* [Python 3.9 installed](https://www.python.org/downloads/).
* [Artillery](https://artillery.io/docs/guides/overview/welcome.html) for load testing of the application.

Start LocalStack Pro with the `LOCALSTACK_API_KEY` pre-configured:

```shell
export LOCALSTACK_API_KEY=<your-api-key>
localstack start
```

> If you prefer running LocalStack in detached mode, you can add the `-d` flag to the `localstack start` command, and use Docker Desktop to view the logs.

## Instructions

You can build and deploy the sample application on LocalStack by running our `Makefile` commands. Run `make deploy` to create the infrastructure on LocalStack. Run `make stop` to delete the infrastructure by stopping LocalStack.

Alternatively, here are instructions to deploy it manually step-by-step.

### Deploy RDS Aurora Cluster with RDS Proxy

**Note:** If you have already provisioned RDS Aurora cluster with RDS Proxy, you can skip 
this step and follow [these steps](#deploy-serverless-workload-using-rds-aurora-as-backend) instead.

This stack will take care of provisioning RDS Aurora PostgreSQL along with RDS proxy fronting it inside
a VPC with 3 private subnet. Required parameters needed by [next step](#deploy-serverless-workload-using-rds-aurora-as-backend)
is also provided as stack output.

```bash
    samlocal build -t rds-with-proxy.yaml --use-container
    samlocal deploy -t rds-with-proxy.yaml --guided
```
### Deploy serverless workload using RDS Aurora as backend

To build and deploy your application for the first time, run the following in your shell:
Pass required parameters during guided deploy.

```bash
    samlocal build --use-container
    samlocal deploy --guided
```

### Installing artillery

We will use [artillery](https://artillery.io/docs/guides/overview/welcome.html) to generate some load towards both the apis. 
Install Artillery via npm:

```
    npm install -g artillery@latest
```

## Load testing

### Checking your installation

If you used npm to install Artillery globally, run the following command in your preferred command line interface:

```
    artillery dino
```

You should see an ASCII dinosaur printed to the terminal. Something like this:

![img.png](images/artillery.png)

### Testing the application

Before starting load testing, make sure `target` in files `load-no-proxy.yml` and  `load-proxy.yml` is update with the 
created HTTP API endpoint. The endpoint is also provided as stack output `ApiBasePath` when 
executing [above steps](#deploy-serverless-workload-using-rds-aurora-as-backend). You can generate load on both the APIs via:

```
    artillery run load-no-proxy.yml
```

```
    artillery run load-proxy.yml
``` 

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.