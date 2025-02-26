# AWS Cloudfront basics
Cloudfront is a mechanism in AWS which replicates mechanism of regional cache, which means that it sets up in predefined regions called edges ( aws datacenters ) cache mechanisms
which will cache data fetched by user, the data might be requested from s3, ec2 and other aws services. Let's say we have our server  deployed in US-east and customers from europe
request same video/image from our server many times a day. By default if users request content for N times, request for this content must go through whole network from one continent
to another one for fetching data N times, and this is also not a great result since data is traversing via public netwrok which might have very high load and even increase delay even more.

## Services/entities cloudFront can be applied to
- AWS S3
- Elastic load balancers ( web servers )
- AWS media channel package endpoint
- AWS mediaStore container endpoint

To solve this problem we can setup cloudfront location in europe where the data will be cached first time its requested from the server of US-east and all subseuquent requests for this
file will be handled from this cache from datacenter which will dramatically decrease time of serving users and will lead to better UX.

## Interesting details 
One of the most interesting points in Cloudfront caching mechanisms is that even the first request, when the actual first copy of the data requested must be fetched from original server
is much faster with cloudfront than it is with regular network, since on the first request user asks cloud front if there is cached data and if not, cloudfront, using aws-s  network will fetch data from s3,ec2 or whatever service was requested.  Since this first network call will also be using aws network and not public network ( which is not fully reliable ) the process will be much faster.

## Creating cloud front service
When creating cloud front service instnace on your account there are quite a few intersting properties to specify
- origin domain: name/identifier of a service you want your cloudfront service to link to ( which service's data you want to cache ).
- origin path: Optional property, specifically for s3 bucket, here you can specify some subdirectory of s3 and cloud front willalways forward traffic for 
fetching from that subfolder. Common practice with s3 is to have different subfolders in same s3 for different env-s. like production and development. 
and use cloudfront to use content from /production subfolder of s3. Since in deveopment mode cloudFront isnot needed.
- S3 bucket access: common pattern with s3 and cloudFront is to make s3 bucket non public and enable Origin access identity ( OAI ) which will only allow
cloud front to access data in s3 buckets. If we don't specify OAI then s3 must be made public.
For this mechanism to work you must create explicitly new Origin access identity or use already precreated one, associate it with cloudFront. When associating
this Origin access identity to cloudFront we also need to update S3 secutiy policies to only allow this OAI. CloudFront creation allows you to automatically
manage this creds. or you can manually do it later.
General mechanism for OAI is to restrict all unsecured/other way access to s3 but access through cloudfront url and cloudfront-s forwawrding mechanism.
- Custom headers: When creating cloudfront you can specify custom headers, which cloudfront will always put when forwarding request to origin service.
We might use this mechanism let's say to only grant read permissions from s3 to requests that have this specified header in them.

## Default cache behaviour configurationI
- Path pattern:
- Automatic Compression
- Viewer protocol policy: HTTP and HTTPS, HTTP -> HTTPS, HTTPs only
- Allowed HTTP method
- Restrict Viewer access

## Common practices
It is a well know fact that AWS CloudFront is widely used for front end static contents, like css html js files, which need fast distribution all aroud the internet.
But while developing architecture for backend systems it might also be a good idea to use cloud front for our services, specifically for ec2 instances serving dynamic and/or static content.
We must notice that by this way we can boost  performance, security, and cost benefits you get when using CloudFront to serve dynamic and/or static assets from Amazon EC2

- Cachable content: While lots of users have their static data on S3 buckets, there are also many usecases when ec2 is running some kind of web server like nginx and that web server is
serving static content, for cases like this cloud front would be a great idea since it will reduce origin’s workload and bandwidth utilization while bringing content closer to viewers. This reduces the latency of serving static content.
- Persistent connections: one cool fact is that Cloudfront maintains persistent connections to origins and maximizes their reuse coefficient. Between cloud front and origin services
traffic is routed via private backbone network of AWS which reduces latency and increases reliabilty.

## Security
Sometimes it's the case that contents of cloudFront must be secured, and not freely accessible to anyone who has an access to a url of content of CloudFront.
In this case best solution already stated by aws CloudFront is to use signedUrls or signedCookies. We create a separate application where user logs in using
credentials and the login service returns either signed cookie or signed url ( this signature is used using public key of keypair let's say cloudfrontKeyPair ).
Then user uses this signed data to access cloudFront data and since cloudFront service has access to cloudFrontKeyPair private key, it can validate request. 

If you use signed urls or signed cookies for cloudFront it secures your urls of cloudfront but if any user still has original S3 or some other origin resource url saved
they can easelly bypass cloudFront security and directly access s3, so we want to avoid this.
1) Create OAI(origin access identity) which is a special cloudfront user and associate it with your distribution 
2) Change permission for S3 bucker or whatever origin resource you to give access to content only through OAI identity.
This way only cloudFront accessed requests will be valid and no bypass methods will be available.

You can create OAI-s from CloudFront dashboard on the left menu bar.

## Diagram of cost mechanism of cloud front, to visualize that it is not expensive mechanism
![diagram cloudfront cost](./diagram.png)
