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

2) Sonra 
