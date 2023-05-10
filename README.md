# AmazonCertificateManagerSetup

Sertifikaları Certbot ve Letsencryp ten AWS e taşımak için aşağıdaki yolları izledik :

Notlarım :

We have 3 ALB. One is for dev, second one is for prod. ALBs are not directly belongs to instances. You need to check the network. Both ALB and EC2 instance are belong to same VPC (same network). For dev, we have to create one ALB, because this dev ALB belong to dev VPC and our dev instance is belong to default VPC. Prod keycloak machine is belong to Prod VPC and dev keycloak machine is belong to default VPC.

This ALB some cost you. We need to confirm manager if he confirm the cost. ***ACM must work with ALB

So due to situation above, we have 2 possible scenario ;

1) dev keycloak machine --> default VPC
   prod keycloak machine --> prod VPC
   
   ECS, EKS,dev ALB are on dev VPC
   prod ALB are on prod VPC
   
   We can delete the dev keycloak machine after taking snopshot, and create new instance under dev VPC. So new machine will be part of this machine.
   
   So keycloak machine should be part of dev VPC. It will some the cost. We dont need to create ALB. At the moment we use two VPC for dev environment. One VPC is default, second one is dev VPC.
   
   **So I understand creating new ALB cost more than moving EC2 Instances from default VPC to dev VPC. Therefore we select the option which is to move EC2 Instances from default VPC to dev VPC.
   
   **Afzal said we need to use wild certificate, but Saddam said we need to spesific certificates instead of using wild certificate since wild certificates causing issues !!
   
   Biz bu 1. scenario yöntemini tercih ettik. Bunun için sırasıyla yapılanlar ;
   
   1) AWS ACM de wilcard certificate request ettik. Sonra Saddam la bunu *.banct-app.com dan idp.banct-app.com a convert ettik. Bunun için mevcuttakini silip yeni bir tane certificate request oluşturmak gerekti. Sonra bu certificates in status unun success olması için bekle. Beklerken  sayfayı refresh et. Sonra bu certificate in içibe girip "Ccreate records in Route 53" ye tıkla. Sonra da "Create Records" butonuna tıkla. Sınra domainin record un oraya (Route 53 ye) eklendiğinden emin ol.

2) Sonra EC2 -> Load balancer dan senin Load balancer ını seç, sonra HTTPs=443 Listener seçip "Add certificate to listener" tıkla. Sonra ordan listeden senin yeni eklediğin certificate ı ekleyip "Include as pending below" butonuna tıkla. Sonra "Add pending Certificates" butonuna tıkla. Sonra Load balancer ın "Listener"  sekmesinde senin certificate inin olduğunu göreceksin. Now this ALB has two certificate . So to summarize, our now certificate is attached to the dev ALB. We can use multiple certificates with one Application Load Balancer. The default certificate for dev ALB was test.banct-api.com. I have added new certificate which is *.banct-api.com (But with Saddam, we removed that wildcard certificate and added idp.banct-api.com).

So in ALB, test.banct-api.com is default certificate
           idp.banct-api.com is listener
           
**Certificates can be attached to Application Load Balancers in ALB in Listener part, we can add more certificates.

**If we use certificate within EC2 instance, if we setup certificate inside the machine, it means we are opening our public ip for the machine. We are direct receiving to the machine. There is no autoscaling. There is no WAF. There is no upper layer AWS resources. It is direct machine and everyone can have the ip can access the machine. Ip is public. Might be IP installed, or might be username and password. But in ALB, ALB is managed service. It keeps all the SSL, all the configs at ingress level.

ALB is like an ingress like we have in kubernetes. As you remember, in ingress, we mention certificates, we mention ports, and everything in ingress level. ALB is similar with ingress. Then ingress will connect to the services. ALB will connect to the EC2 machine.

ingress <--> service

**If I run kubernetes and I want to access any service on browser, we need to create one ingress and in ingress, I need to mention everything, domain, path, port, everything, liveness and readiness.And that ingress will be responsible for traffic. It will go to the ingress and ingress will go to the service, and will do rest of the work. Similar concept in ALB. We hae ALB which is ingress, and it is responsible to connect to the EC2 machine with help of target group. It will not expose any ip to the Internet. No one knows that EC2 machine is running behing the ALB. Serverless environment is running behind ALB. Any ECS or EKS running behind the ALB. So in front of ALB, the infrastructure will be high. Thats why people prefer to use ALB instead of directly machine.


3) EC2 makinesinin snopshot ını oluşturuyoruz. 

EC2 --> Actions --> Image and Templates --> Create Image

Image ı oluştururken image name yaz, aynısını imae description a kopyala Sonra aşağı kaydırıp key ekle, value ekle, key kısmına "name" yaz, value kısmına image ismine ne yazdıysan onu yaz. Sonra snopshot ını oluşturduk. Bu oluşan snopshot AMIs altında görülür. Bunun status u pending den "Available" a geçmesi için bekledik.

Creating new ALB cost much money. We just added one certificate which cost little. Creating new instance does not cost much money.

**In our VPC, we have 4 subnet, two subnet are private, two subnet are public. They are in two availability sone. One zone is eu-west-2a, second zone is eu-west-2b. Public subnets have route to public route, and it is going to the Internet. And for the private we have. Either we can use one Route table as well for the Private Subnet as well. Because having two Route table is not good practice. But we can not change it now. It is not possible to change.

4) Snopshot (AMI) oluştuktan sonra yeni dev keycloak instance ı oluşturduk. Yeni EC' nun instance type = t2. medium

Keypair = Bir önceki makine nin keypair ini seçtik. Network Settings i editleyip VPC yi default VPC den dev VPC ye değiştirdik. Subnet , public subnet seçtik. Yeni dev keycloak instance ı oluştururken status u "active" hale geçen ilgili ilgili AMI yi seçip "Create Instance From AMI" butonuna tıkladık. Sonra instance a isim verdik. Sonra inbound security group rule kısmında ssh ın altındaki rule un source type ı "Anywhere" den custom olarak değiştirdik. Sonra security group name i kendimize göre güncelledik. Sonrada launch instance butonuna bastık. Sonra instance ın ayağa kalkmasını bekledik. State inin "Pending" den "Active" e geçmesini bekledik.

**While creating EC2 machine, for subnet part, we have selected public subnet. Because if we have select private subnet, we wont be able to ssh machine without VPN.

5) Sonra yeni oluşan EC2 makinesinin public ip sine erişiliyormu diye baktık. Erişilmiyordu.

**Eğer ALB ve EC2 farklı VPC delerse onlar birbirine bağlı değildir diyoruz. Ama EC2 yu dev VPC ye taşıyınca ikisi aynı network dedir ve birbirine bağlıdır diyoruz. They were not used to connect each other.

**ALB contains SSL. So we need ALB for ACM. If we need to use ACM (Amazon Certificate Manager), we must use ALB.

6) Sonra Afzal yeni makinaya SSG yapıp nginx , restart etti.
  
 Sonra yeni makinanın security group unu kontrol edip inbound rule ekledi. SSH 22, TCP 8443, HTTP 80, HTTPs 443 ekledik.
 
 Sonra EC2 makinasının public ip sine yeniden erişmeye çalıştık. Sonra nginx i restart edince sorun düzeldi. Keycloak instance public ip sine erişebildik.
 
 7) Then we need to create target group, target group will do association between ALB and the machine. So target group is the bridge. It will connect Load Balancer to the machine. Target group u oluştururken port u TCP seçtik ve doğru VPC yi seçtik.

Target group oluştururken

       - Target group name yazdık.
       - Protocol u TCP yaptık.
       - Doğru VPC yi seçtik.
       
Target group oluşturduktan sonra "Associate with an Existing Load Balancer" seçtik. Çünkü "Non associated" olarak görünüyordu.  Bizde "Associate with an Existing Load Balancer" ı seçip ordanda kendi Load balancer ımızı seçtik. Sonra bu Load balancer ın rules larına gidip ordan HTTPS=443  rules içerisinde kendi rule umuzu ekledik.

Then target group was not healthy, and we fixed this issue. To fix tagret group not healthy issue, we changed health check setting from part 443 to port 80.

8) Sonrada Route 53 den bizim domain için alias lı record ekledik. Bu record u eklerken

   - Record type = A
   - Route traffic to = Alias to Application and Classic Load Balancer
   - Europe (London) eu-west-2
   - Ve de dual stack DNS name seçtik. Bunu sonundaki 929 rakamından Load Balancer a eşleştirdik. Load Balancer DNS name de yazana göre bunu seçtik.

Sonra bu record un status un pending den active e geçmesi için bekledik. 
On Route 53, I just associate this domain with ALB. Alias is masking

**ALB hiding everything in front of internet
**Type A can be ip or can be domain name. CNAME is text record, can be verified.
**For TXT, for nsrecord, for email verification belongs to domain. We choose CNAME mostly, for SSL, we choose CNAME.

A record can host the host name to one or more ip address. But CNAME can maps hostname to another hostname. If we have two domain and we want to map, we can use CNAME
 if we have one domain and one ip we want to map, we have to use A record.
 
 **A record maps a hostname to one or more IP address while the CNAME record maps a hostname to another hostname.
 
 Port 80, nginx is running, port 443 keycloak is running. If we choose port 443 in healthcheck, it is not responding to target group. Because application is running internally.
 
 Ourt healthcheck is working over nginx (port 80). Our traffic is coming from port 443, and it is going to 443 to 80.
 
 **/Port 443 is not responding to the target group. Therefore Afzal updated port 443 to port 80 in heatlth check setting. On port 80, nginx is running.
 
 **Nginx configine bakarsak, port 443 is not doing anything directly. It is proxy port.. It is related to SSL, proxy etc. Port 443 is not directly doing anything. This port is not available directly. It is the proxy port. It is a way to access port 443 only with proxy.
 
 **Sonra Afzal la görüşmelerimizden sonra ALB ye gidip port 80 yi port 443 e yönlendiren bir listener rule u ekledik.
 
 **If anyone wants to access port 80, it will automatically redirect to port 443. We are not using nginx anymore. We are using ALB instead of nginx for routing purposes.
 
To summarize, ALB is amazing. It is useful for autoscaling. With target group, we can use 5 machine.
   






















