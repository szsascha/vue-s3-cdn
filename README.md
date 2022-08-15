# Vue S3 CDN

Fast delivery of static Vue applications via VitePress, AWS S3 and Cloudflare CDN to any location in the world.

## About this solution

During the preparation of a sideproject I've found that the potential usergroup is spread all over the world. Starting with the most Google search queries from the Philippines and India. Closely followed by the USA and Canada and also many searches from Europe.
I decided to target all countries at once since the project was very small and many people in these regions are english speaking.

So I needed a solution to target all regions with a proper loading time. Consequently I was in the need to have servers all over the world. A hosting just in the US or Europe would lead to pagespeeds higher than 4 seconds in Asia.

### Criteria for a possible solution

I've set the following criteria for a possible solution since it was only a small sideproject without intention to generate profit:
1. Fast delivery to all places around the world
2. Cost-effective
3. Good SEO potential
4. Website delivery as HTTPS
5. Rapid development workflow (esp. with GitHub and gitflow)
6. Easy and quick to set-up
7. Bulletproof and reliable (I don't want to deal with exotic problems due to exotic solutions)
8. I wanted to use an existing domain
9. Last but not least: I always want to learn about new things and technologies. So I would like to use a few technologies which are new to me.

### The solution

So I came up with the Idea of using an AWS S3 bucket with configuration for static website delivery in combination with Cloudflare as a proxy and VitePress for static site generation of a Vue codebase.

After a bit of googling I found out that S3 in combination with Cloudflare are used by many others. So I gave it a try.

### The implementation

The implementation was really easy, quick and cost-effective. Even if I haven't worked with S3, Cloudflare and VitePress before I was able to create a dummy implementation on a single afternoon. So I learned a lot that day.

The costs of S3 are a few pennies, if at all. The free plan of Cloudflare is absolute enough for this use case. And the rest of the used technologies are also free or even open source.

Cloudflare provides a free CDN with hundreds of edge servers all over the world. It even reduces the cost of S3 by caching the requests. Cloudflare also provides many more features like DDoS protection in its free plan.

It also provides an HTTPS configuration incl. certificate by default. An existing domain can be used by just changing the nameservers to Cloudflares.

I choose VitePress because it's a good way to generate static (even minimized) pages from a Vue codebase. Static pages are necessary for SEO reasons. Just Vue will result in almost uncrawlable site.
It can be easy managed in a GitHub Repo and it's also really easy to push changes from the master/main branch automatically via GitHub Actions to the S3 Bucket.

## Requirements
1. yarn installed on your local machine
2. GitHub Account and Repository (free plan and privare repository is enough)
3. git client installed and configured for GitHub on your local machine (GitHub client is enough)
4. Domain (be sure that you're able to change the nameservers)
5. Cloudflare Account (free plan is enough)
6. AWS account 

## Setup

I'll explain in the following how this solution can be set up and include some sample's so that it works immediately. In this repository you can also find all files which are deployed from a git repository.

But first of all we've to decide which domain we want to use. For this part I choose example.com. We also create a redirect from www.example.com to example.com.
Please replace example.com in your usecase with your domain.

### Create and configure an S3 Bucket for static site usage

At first we've to create a new bucket for our example.com:

1. Login to AWS and open the S3 Management Console
2. Create a new bucket
3. Choose your domain name as Bucket-Name (i.e. example.com)
4. Any other configuration is at this point optional
5. Create your new bucket

Now we've to configure static website hosting for our bucket:

1. Go to your newly created bucket and enable the static hosting in the properties
2. Configure *index.html* as indexdocument
3. Save the static website hosting configuration

As mentioned at the beginning we're now creating our redirection from www.example.com to example.com:

1. Create a new bucket *www.example.com* as you've done for *example.com*
2. Go again to the newly created bucket and enable the static hosting in the properties
3. But this time don't leave the default configuration. Choose "redirection" as hosting type instead.
4. Type *example.com* as hostname and select https protocol

Now we've created our buckets. But no one have access to the buckets. In the next step we've to ensure that only Cloudflare gets access to the buckets because only the traffic coming from Cloudflare will have access to the sites. We do this for different reasons. One reason is to prevent double content for SEO.

1. Open your *example.com* bucket configuration
2. Go to permissions and edit the policy
3. Disable blocking of all public access. We now have to *allow* the incoming traffic we want.
4. Put the policy JSON below into the policy field and save it after you've change the domain name.
5. Repeat that for *www.example.com*. Don't forget to add *www.* in front of the *example.com* url in the policy.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example.com/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": [
                        "173.245.48.0/20",
                        "103.21.244.0/22",
                        "103.22.200.0/22",
                        "103.31.4.0/22",
                        "141.101.64.0/18",
                        "108.162.192.0/18",
                        "190.93.240.0/20",
                        "188.114.96.0/20",
                        "197.234.240.0/22",
                        "198.41.128.0/17",
                        "162.158.0.0/15",
                        "104.16.0.0/13",
                        "104.24.0.0/14",
                        "172.64.0.0/13",
                        "131.0.72.0/22",
                        "2400:cb00::/32",
                        "2606:4700::/32",
                        "2803:f800::/32",
                        "2405:b500::/32",
                        "2405:8100::/32",
                        "2a06:98c0::/29",
                        "2c0f:f248::/32"
                    ]
                }
            }
        }
    ]
}
```
(The subnets are from Cloudflare)

### Configure Cloudflare

1. Add *example.com* as new site to your Cloudflare account
2. Select free plan
3. Check if there are already DNS entries for the root *example.com* and *www*. Delete them if they already exists. But be sure that you're really not needing them.
4. Create a new CNAME root entry for your root *example.com* and *www*. Point them to the URL you can find in your AWS configuration properties. For *example.com* and *www.example.com*. (i.e. example.com.s3-website-eu-west-1.amazonaws.com)
5. Change your domain's nameserver like its shown by Cloudflare
6. Enable "Always Use HTTPS"
7. **Don't** enable auto minify since Vue needs comments in the HTML file to work
8. Change "SSL/TLS" from "Full" to "Flexible"

You can find further information on [Cloudflare's official tutorial](https://support.cloudflare.com/hc/en-us/articles/360037983412-Configuring-an-Amazon-Web-Services-static-site-to-use-Cloudflare).

### Create the VitePress codebase

For this example we're create the VitePress codebase like in the [getting started](https://vitepress.vuejs.org/guide/getting-started.html) described.

1. yarn init
2. yarn add --dev vitepress vue
3. mkdir docs
4. echo '# Hello World\n[Go to another page](page.md)' > docs/index.md
5. echo '# Another page\nJust to be sure that references are working correctly.' > docs/page.md
6. Add scripts to package.json like you find below
7. Run local dev server yarn docs:dev
8. Open http://localhost:3000/
9. Push the files to the master branch of your GitHub repository

```
{
  ...
  "scripts": {
    "docs:dev": "vitepress dev docs",
    "docs:build": "vitepress build docs",
    "docs:serve": "vitepress serve docs"
  },
  ...
}
```

### Configure the GitHub actions pipeline

Place the following contents into your .GitHub/workflows/build.yml.
Mind to change the region and s3://example.com to your domain.

Add the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY secrets to your GitHub repository. You can create them in your AWS console.

```
name: S3 Deploy
on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@master

      - name: Cache node modules
        uses: actions/cache@v1
        with:
          path: node_modules
          key: ${{ runner.OS }}-build-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.OS }}-build-
            ${{ runner.OS }}-
      - name: Install
        run: yarn install

      - name: Build
        run: yarn docs:build

      - name: Deploy
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws s3 cp \
            --recursive \
            --region eu-west-1 \
            docs/.vitepress/dist s3://example.com
```

## A few things to keep in mind

Of all solutions I considered, this one fets the best to my project. But all that glitters is not gold. This solution also have some disadvantages you've to be aware of. In my current case I was able to see over it.

Just to share a few thoughts with you and hopefully prevent you from falling into a hole:
- This solution doesn't provide a real end to end HTTPS encryption. The traffic is encrypted from your browser to Cloudflare. But isn't encrypted between Cloudflare and AWS. That makes a man in the middle attack potentially possible. But you've to decide yourself how critical this scenario is for your usecase. You can prevent this by using AWS Cloudfront. There are also [different approaches](https://www.chrislockard.net/posts/secure-cloudflare-s3/). But again. You have to decide for yourself.
- Depending on where you live you maybe have to be aware of privacy regulations. Like EU's GDPR. This includes the not encrypted data transfer between Cloudflare and S3 and also the regulation texts you need on your website / app. There should be own segments for Cloudflare and AWS.
- Where do you put your backend code? I don't have backend code for this part of my sideproject.

## Disclaimer

This repository is just an example how it could work. Please think for yourself twice before you implement things you've found on the internet. As I mentioned, this way isn't 100% safe and you should actively decide if it is working safely for your usecase.