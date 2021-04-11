# DNS setting using Route 53 and freenom

## Conditions
- AWS Route 53
- [freenom](https://my.freenom.com/clientarea.php)


## 1. Register Domain name on Freenom

- Register a Domain at Freenom

  see [完全無料の独自ドメインを取得できるFreenomの使い方](https://www.lancork.net/2019/05/how-to-use-freenom-free-domain/)

- Create Hosted Zone at AWS Route 53
  - Move to Route 53 Service on the Managed Console.
  - Click [Create hosted zone] 
  - Put the domain name that previously was set on Freenom in `domain name` text field

    `ex.` domain name: www.seungxyz.cf, sub.seungxyz.cf

- Set nameservers the regsited domain on freenom
  - Click links with following step [Services] -> My Domains
  - Click [Manage Domain] at your domain of domain list.
  - Select [Management Tools] tab and then [Nameservers].
  - Select the [Use custom nameservers] checkbox
  - Set Nameserver1 to 4 from the hosted zone at Route53 you registered.

    see [AWSでWebサイトをHTTPS化 その5：CloudFront(+証明書)→S3編](https://recipe.kc-cloud.jp/archives/11256)

## 2-1. Setup DNS as the Route53 if EIP will be set on EC2 instance or external server
- Finish step 1(registering DNS)

- Get EIP(Elastic IP)
  - Move to EC2 service and click [Allocate new address]
  - Click Actions and [Associate address] at EIP

    see [Attach Elastic IP to EC2 Instance](https://avinton.com/academy/route53-dns-vhost/)

- Create Record
  - Move go [Go to Record Sets] from [Hosted Zone] link on Rout 53
  - Select [A - IPv4 address] at type
  - Put EIP at value


## 2-2. Setup DNS with the Route53 when using S3 and Cloudfront

- Finish step 1(registering DNS)

- Create and Import ACM (AWS Certificate Manager)
  - Go to ACM and change region into `Virginia`[us-east-1] <- it's necessary
  - Click [Get start] -> [Request certificate]
  - Put `domein name` that created at step 1 into [domain name] field 

    `ex.` `www.seungxyz.cf`
  - Add other name and put `sub domain name`. 

    `ex.` `sub.seungxyz.cf`
  - Go to [next] and check [DNS validation] then go to [next]
  - Add tag and go to [next]
  - Click [Confirm and request]
  - save the CNAMES from [Export DNS configuration to a file]
  - You can see two domains (`www.seungxyz.cf`, `sub.seungxyz.cf`) and click both of domain's [Create record in Route 53]
  - Confirm to change status from [pending validation] to [issued] in a while.

    see [SSL/TLS 証明書を DNS 検証を使ってリクエストする](https://aws.amazon.com/jp/blogs/news/easier-certificate-validation-using-dns-with-aws-certificate-manager/)


- Upload resources on S3

    see [AWSでWebサイトをHTTPS化 その5：CloudFront(+証明書)→S3編](https://recipe.kc-cloud.jp/archives/11256)

- Create Distribution on CloudFront
  - Set data from following [AWSでWebサイトをHTTPS化 その5：CloudFront(+証明書)→S3編](https://recipe.kc-cloud.jp/archives/11256) at CloudFront section

  - put sub domain name into `Alternate Domain Names(CNAMEs)`
    
    `ex.` `sub.seungxyz.cf`
  - put a file name that you uploaded the html file or other resource on S3 bucket into `Default Root Object`

    `ex.` If you uploaded the name `https://seung-dns-test.s3-ap-northeast-1.amazonaws.com/dashboard.html` on S3 bucket, put the name.

  - Confirm to change Status into `Deployed` after created the Distributions (You need to wait for a while)

  - Create A record on Route 53
    - Click to `seungxyz.cf` Hosted zone
    - Click [Go to Record Sets]
    - Put prefix of sub domain name that you set same as the name at CloudFront Distribution into [Name]
      
      `ex.` `sub` (prefix of sub.seungxyz.cf)
    - Select [A - IPv4 address] at type
    - Change Alias `Yes`
    - Select CloudFront Distribution Name you created and Click [Create]


  - Check the resource from the browser
    - Go to `https://sub.seungxyz.cf/dashboard.html`
