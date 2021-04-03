---
layout: post
title: Serverless quick tip&#58; Use multi-region, active-active architectures for performance and resiliency
categories: [aws, serverless]
---

This post is a quick rebuttal to some tweets I saw this past week advising people to avoid trying multi-region architectures because they are too much work and will kill innovation in other areas. As a blanket statement, this is bad advice. Yes, I understand there are systems that don't lend themselves to easily doing multi-region. In those cases it probably doesn't make sense. But there are many systems that **do** lend themselves to this sort of architecture without much work – and the payoff can be big.

Of couse, I'm thinking of serverless here. It's another in a long list of advantages serverless brings to the table. One of the multi-region, active-active services I built at my current job gets about 8 million requests per month. It has not only never gone down in the nearly two years it has been in service, it has also provided big performance benefits. Our target audience is the US. However, we get signficant (and growing) traffic from Asia. Prior to doing multi-region, those users were seeing some pretty lousy Time To First Byte numbers. And the cost to do multi-region for this service was trivial. The increased cost was mostly around syncing data with DynamoDB global tables.

There are a number of tutorials I will link to below, but the basic steps to get a multi-region, active-active architecture set up on AWS are:
1. Deploy your service/app to the desired regions.
2. Depending on your service, set up [DynamoDB global tables](https://aws.amazon.com/dynamodb/global-tables/) so your data is co-located with your application.
3. If you're using API Gateway, configure the [custom domain name](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-custom-domain-names.html) (e.g. api.example.com) in each region.
4. Configure [health checks in Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/health-checks-creating.html) for each region. You won't use the custom domain name here, but rather the endpoint provided by API Gateway (or whatever other service you are using).
5. Add the [latency-based DNS records](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html#routing-policy-latency) in Route 53. Each record will be for the same domain but will point to a different origin URL for each region and will use the health check you set up for that region.

The first time you do this it may feel a little clunky, but the next time will go much faster. A couple resources with more details are [here](https://medium.com/dazn-tech/serverless-aws-latency-routing-d11d9160699e) and [here](https://www.edureka.co/blog/amazon-route-53/). Another option that you may want to check out is [Cloudflare Load Balancing](https://developers.cloudflare.com/load-balancing/). In general, I found it easier to set this up in Cloudflare, but we ultimately weren't able to use Cloudflare for this due to the very high cost of cache invalidations via the API. Super expensive!

Anyway, if you have stumbled upon this I hope you have found it useful and will give it a try. When I saw the tweets that were critical of trying this type of architecture I immediately worried it would put new cloud engineers off the path to building highly available and performant services. If you have any questions (or complaints!) about this post, you can [DM me on Twitter](https://twitter.com/bodhiprice) or email me at [jprice@hey.com](mailto:jprice@hey.com).
