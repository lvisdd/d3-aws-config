Title: AWS ConfigをD3.jsで可視化する
Subtitle: AWS ConfigをD3.jsで可視化!
Author: 加納 邦彦
Author(romaji): Kunihiko Kanou
Twitter: @k_kanou

# AWS Config とは

　[AWS Config](https://aws.amazon.com/jp/config/) は、セキュリティとガバナンスを可能にする構成変更の通知、構成履歴、AWS リソースのインベントリーをお客様へ提供する完全マネージド型のサービスです。AWSリソースの構成や設定変更の履歴を監査・可視化することが出来ます。

> AWS再入門 AWS Config編
> http://dev.classmethod.jp/cloud/aws/cm-advent-calendar-2015-aws-re-entering-aws-config/

> 「AWS Configを使ったAWS環境の見える化」
> http://www.slideshare.net/morisshi/develipersio-2016-e1-aws-configaws

# AWS Config Partner について

　AWS Config Partnerに登録されている有料のサービスを活用することもできます。ただ、有料のサービスのため、個人では気軽に試しにくいと思います。そのため、今回はD3.jsで可視化を試みます。

> AWS Config Partners
> https://aws.amazon.com/config/partners/

# D3.js について

　[D3.js](https://d3js.org/) は、JavaScriptの可視化ライブラリです。CSVやJSON形式のデータを読み込み、HTMLやSVG形式で可視化をすることができます。

> D3 gallery
> https://github.com/d3/d3/wiki/Gallery

# AWS Configのサンプルデータ について

　AWS Configのスナップショットのサンプルは以下から取得しました。

> Example Configuration Snapshot from AWS Config
> http://docs.aws.amazon.com/ja_jp/config/latest/developerguide/example-s3-snapshot.html

# AWSのアイコン について

　AWSのアイコンは以下から取得しました。

> AWS Simple Icons for Architecture Diagrams
> https://aws.amazon.com/jp/architecture/icons/

# ディレクトリ構成 について

　今回用意する資材は、以下のディレクトリ構成とします。

```
C:\work\d3-aws-config
│  index.html
│  
├─images
│  └─AWS_Simple_Icons_EPS-SVG
│      ├─Compute
│      │      Compute_AmazonEC2_instance.svg
│      │      
│      └─General
│              General_AWScloud.svg
│              General_virtualprivatecloud.svg
│              
└─json
        aws_config.json

```

# AWS ConfigをD3.jsで可視化した例 について

　以下が表示例です。動作確認は、FireFoxでしました。

![sample.jpg](sample.jpg)

# Google Chromeで表示する場合の注意点 について

　jsonの読み込みに「file://」を使っているため、Google Chromeでは以下の警告が出力され、ローカルのファイルを開くことができません。

```
d3.v3.min.js:1 XMLHttpRequest cannot load file:///C:/work/d3-aws-config/json/aws_config.json. Cross origin requests are only supported for protocol schemes: http, data, chrome, chrome-extension, https, chrome-extension-resource.
```

　コマンドプロンプトを「管理者として実行」し、「--allow-file-access-from-files」オプションをつけることで表示できると思います。

```
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --allow-file-access-from-files C:\work\d3-aws-config\index.html
```

# サンプルコード について

```html:index.html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="utf-8">
  <title>D3.js x AWS Config</title>
</head>
<body>
  <script src="http://d3js.org/d3.v3.min.js" charset="utf-8"></script>
  <script>
    var width = 1000;
    var height = 800;
    var image_size = 100;
    var text_size = image_size / 2;
    var x = 0;
    var y = 0;
    var item_num = 0;

    // SVG
    var svg = d3.select("body").append("svg")
                               .attr({ width: width,
                                       height: height
                                     });

    // AWS Config
    // 
    // Example Configuration Snapshot from AWS Config
    // http://docs.aws.amazon.com/ja_jp/config/latest/developerguide/example-s3-snapshot.html

    d3.json("./json/aws_config.json", function(data){
    
      // AWS Icon
      // 
      // AWS Simple Icons for Architecture Diagrams
      // https://aws.amazon.com/jp/architecture/icons/

      var aws = svg.append("g")
                aws.append("image")
                   .attr({ "class": "image",
                           "xlink:href":"images//AWS_Simple_Icons_EPS-SVG/General/General_AWScloud.svg",
                         });

          // accountId
          x = x + image_size;
          y = y + image_size / 2;

          aws.append("text")
             .text("accountId: " + data.configurationItems[0].accountId)
             .attr({ "x": x,
                     "y": y,
                   });

          aws.append("rect")
             .attr({ "class": "rect",
                     "width": width,
                     "height": height,
                   });

      y = y + image_size / 2;
      
      // VPC Icon
      item_num = item_num + 1;

      var vpc = aws.append("g")
                  // y = y + image_size / 2;
                  vpc.append("image")
                     .attr({ "class": "image",
                             "xlink:href":"images//AWS_Simple_Icons_EPS-SVG/General/General_virtualprivatecloud.svg",
                             "x": x,
                             "y": y
                           });

      // AWS::EC2::Instance
      for(var i=0; i<data.configurationItems.length; i++){
        if (data.configurationItems[i].resourceType == "AWS::EC2::Instance"){

          x = x + image_size;
          y = y + image_size / 2;

          vpc.append("text")
             .text("vpcId: " + data.configurationItems[i].configuration.vpcId)
             .attr({ "x": x,
                     "y": y,
                   })

          vpc.append("rect")
             .attr({ "class": "rect",
                     "width": width - x * item_num,
                     "height": height - y * item_num,
                     "x": image_size * item_num,
                     "y": image_size * item_num,
                   });

          // instanceId
          // instanceType
          // subnetId
          // privateIpAddress
          // publicIpAddress
          // deviceName
          // ebsOptimized
          item_num = item_num + 1;

          ec2_items = ["instanceId", "instanceType", "subnetId", "privateIpAddress", "publicIpAddress", "deviceName", "ebsOptimized"];

          var ec2 = vpc.append("g")

                    // EC2 Icon
                    y = y + image_size / 2;

                    ec2.append("image")
                       .attr({ "class": "image",
                               "xlink:href":"images//AWS_Simple_Icons_EPS-SVG/Compute/Compute_AmazonEC2_instance.svg",
                               "x": x,
                               "y": y
                             });

                    x = x + image_size;
                    // y = y + image_size / 2;

                    for(var j=0; j<ec2_items.length; j++){
                      if (ec2_items[j] == "deviceName") {
                        // deviceName
                        var deviceName = [];
                        for(var k=0; k<data.configurationItems[i].configuration.blockDeviceMappings.length; k++){
                          deviceName.push(data.configurationItems[i].configuration.blockDeviceMappings[k].deviceName)
                        }

                        y = y + image_size / 2;
                        ec2.append("text")
                           .text("deviceName: " + deviceName.join(", "))
                           .attr({ "x": x,
                                   "y": y,
                                 })
                      } else {
                        y = y + image_size / 2;
                        ec2.append("text")
                           .text(ec2_items[j] + ": " + data.configurationItems[i].configuration[ec2_items[j]])
                           .attr({ "x": x,
                                   "y": y,
                                 })
                      }

                    }

                    ec2.append("rect")
                       .attr({ "class": "rect",
                               // "width": width - 200 * item_num,
                               "width": width - x * item_num,
                               "height": text_size * (ec2_items.length + 1),
                               "x": image_size * item_num,
                               "y": image_size * item_num,
                             });
          }
      }

      d3.selectAll(".image").attr({ "opacity": 1,
                                    "width"  : image_size,
                                    "height" : image_size,
                                  });
      
      d3.selectAll(".rect").attr({ "stroke": "black",
                                   "stroke-width": 0.5,
                                   "fill": "#000",
                                   "fill-opacity": 0,
                                   "rx": 10,
                                   "ry": 10,
                                 });

    });
  </script>
</body>
</html>
```

```json:aws_config.json
{
    "fileVersion": "1.0",
    "requestId": "asudf8ow-4e34-4f32-afeb-0ace5bf3trye",
    "configurationItems": [
        {
            "configurationItemVersion": "1.0",
            "resourceId": "vol-ce676ccc",
            "arn": "arn:aws:us-west-2b:123456789012:volume/vol-ce676ccc",
            "accountId": "12345678910",
            "configurationItemCaptureTime": "2014-03-07T23:47:08.918Z",
            "configurationStateID": "3e660fdf-4e34-4f32-afeb-0ace5bf3d63a",
            "configurationItemStatus": "OK",
            "relatedEvents": [
                "06c12a39-eb35-11de-ae07-adb69edbb1e4",
                "c376e30d-71a2-4694-89b7-a5a04ad92281"
            ],
            "availibilityZone": "us-west-2b",
            "resourceType": "AWS::EC2::Volume",
            "resourceCreationTime": "2014-02-27T21:43:53.885Z",
            "tags": {},
            "relationships": [
                {
                    "resourceId": "i-344c463d",
                    "resourceType": "AWS::EC2::Instance",
                    "name": "Attached to Instance"
                }
            ],
            "configuration": {
                "volumeId": "vol-ce676ccc",
                "size": 1,
                "snapshotId": "",
                "availabilityZone": "us-west-2b",
                "state": "in-use",
                "createTime": "2014-02-27T21:43:53.0885+0000",
                "attachments": [
                    {
                        "volumeId": "vol-ce676ccc",
                        "instanceId": "i-344c463d",
                        "device": "/dev/sdf",
                        "state": "attached",
                        "attachTime": "2014-03-07T23:46:28.0000+0000",
                        "deleteOnTermination": false
                    }
                ],
                "tags": [
                    {
                        "tagName": "environment",
                        "tagValue": "PROD"
                    },
                    {
                        "tagName": "name",
                        "tagValue": "DataVolume1"
                    }
                ],
                "volumeType": "standard"
            }
        },
        {
            "configurationItemVersion": "1.0",
            "resourceId": "i-344c463d",
            "accountId": "12345678910",
            "arn": "arn:aws:ec2:us-west-2b:123456789012:instance/i-344c463d",
            "configurationItemCaptureTime": "2014-03-07T23:47:09.523Z",
            "configurationStateID": "cdb571fa-ce7a-4ec5-8914-0320466a355e",
            "configurationItemStatus": "OK",
            "relatedEvents": [
                "06c12a39-eb35-11de-ae07-adb69edbb1e4",
                "c376e30d-71a2-4694-89b7-a5a04ad92281"
            ],
            "availibilityZone": "us-west-2b",
            "resourceType": "AWS::EC2::Instance",
            "resourceCreationTime": "2014-02-26T22:56:35.000Z",
            "tags": {
                "Name": "integ-test-1",
                "examplename": "examplevalue"
            },
            "relationships": [
                {
                    "resourceId": "vol-ce676ccc",
                    "resourceType": "AWS::EC2::Volume",
                    "name": "Attached Volume"
                },
                {
                    "resourceId": "vol-ef0e06ed",
                    "resourceType": "AWS::EC2::Volume",
                    "name": "Attached Volume",
                    "direction": "OUT"
                },
                {
                    "resourceId": "subnet-47b4cf2c",
                    "resourceType": "AWS::EC2::SUBNET",
                    "name": "Is contained in Subnet",
                    "direction": "IN"
                }
            ],
            "configuration": {
                "instanceId": "i-344c463d",
                "imageId": "ami-ccf297fc",
                "state": {
                    "code": 16,
                    "name": "running"
                },
                "privateDnsName": "ip-172-31-21-63.us-west-2.compute.internal",
                "publicDnsName": "ec2-54-218-4-189.us-west-2.compute.amazonaws.com",
                "stateTransitionReason": "",
                "keyName": "configDemo",
                "amiLaunchIndex": 0,
                "productCodes": [],
                "instanceType": "t1.micro",
                "launchTime": "2014-02-26T22:56:35.0000+0000",
                "placement": {
                    "availabilityZone": "us-west-2b",
                    "groupName": "",
                    "tenancy": "default"
                },
                "kernelId": "aki-fc8f11cc",
                "monitoring": {
                    "state": "disabled"
                },
                "subnetId": "subnet-47b4cf2c",
                "vpcId": "vpc-41b4cf2a",
                "privateIpAddress": "172.31.21.63",
                "publicIpAddress": "54.218.4.189",
                "architecture": "x86_64",
                "rootDeviceType": "ebs",
                "rootDeviceName": "/dev/sda1",
                "blockDeviceMappings": [
                    {
                        "deviceName": "/dev/sda1",
                        "ebs": {
                            "volumeId": "vol-ef0e06ed",
                            "status": "attached",
                            "attachTime": "2014-02-26T22:56:38.0000+0000",
                            "deleteOnTermination": true
                        }
                    },
                    {
                        "deviceName": "/dev/sdf",
                        "ebs": {
                            "volumeId": "vol-ce676ccc",
                            "status": "attached",
                            "attachTime": "2014-03-07T23:46:28.0000+0000",
                            "deleteOnTermination": false
                        }
                    }
                ],
                "virtualizationType": "paravirtual",
                "clientToken": "aBCDe123456",
                "tags": [
                    {
                        "key": "Name",
                        "value": "integ-test-1"
                    },
                    {
                        "key": "examplekey",
                        "value": "examplevalue"
                    }
                ],
                "securityGroups": [
                    {
                        "groupName": "launch-wizard-2",
                        "groupId": "sg-892adfec"
                    }
                ],
                "sourceDestCheck": true,
                "hypervisor": "xen",
                "networkInterfaces": [
                    {
                        "networkInterfaceId": "eni-55c03d22",
                        "subnetId": "subnet-47b4cf2c",
                        "vpcId": "vpc-41b4cf2a",
                        "description": "",
                        "ownerId": "12345678910",
                        "status": "in-use",
                        "privateIpAddress": "172.31.21.63",
                        "privateDnsName": "ip-172-31-21-63.us-west-2.compute.internal",
                        "sourceDestCheck": true,
                        "groups": [
                            {
                                "groupName": "launch-wizard-2",
                                "groupId": "sg-892adfec"
                            }
                        ],
                        "attachment": {
                            "attachmentId": "eni-attach-bf90c489",
                            "deviceIndex": 0,
                            "status": "attached",
                            "attachTime": "2014-02-26T22:56:35.0000+0000",
                            "deleteOnTermination": true
                        },
                        "association": {
                            "publicIp": "54.218.4.189",
                            "publicDnsName": "ec2-54-218-4-189.us-west-2.compute.amazonaws.com",
                            "ipOwnerId": "amazon"
                        },
                        "privateIpAddresses": [
                            {
                                "privateIpAddress": "172.31.21.63",
                                "privateDnsName": "ip-172-31-21-63.us-west-2.compute.internal",
                                "primary": true,
                                "association": {
                                    "publicIp": "54.218.4.189",
                                    "publicDnsName": "ec2-54-218-4-189.us-west-2.compute.amazonaws.com",
                                    "ipOwnerId": "amazon"
                                }
                            }
                        ]
                    }
                ],
                "ebsOptimized": false
            }
        }
    ]
}
```