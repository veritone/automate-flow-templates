# Overview

This flow template is intended to serve as an example of how to currently create flow engines that themselves make API requests to aiWARE for running engine processing on the media specified in the flow logic. 

## Template flow

1. Expand the collapsed code section
1. Copy the entire JSON in the section
1. Navigate to an existing Flow Engine Detail page or create a new Engine using the _New Engine_ creation wizard
1. Once in the Engine Detail page, click the **Create in Automate** button
1. When the Automate Studio editor loads, click on the hamburger menu and select the **Import** option
1. Paste the JSON on your clipboard into the flow and click Import
1. That's it, you now have imported this template to experiment with in your flow engine!

<details>
<summary>Expand for flow definition </summary>

```
[{"id":"e8111db7.09df3","type":"subflow","name":"aiWARE Poll Job (BETA)","info":"# Overview\nThis subflow encapsulates the polling pattern that accepts a `JobId` and polls the Job status\n\n## Input\n - `msg.jobId` - expects a valid Id of a Job created in your Veritone Organization\n\n## Output\n 1. The top port outputs the `msg.payload` and stores the results of the job query in JSON format\n 2. The bottom port outputs *failed* or *aborted* jobs in the `msg.payload` property\n\n## Beta Tag\n - Does *not* gracefully handle errors\n - Does *not* handle edge cases very well\n","category":"","in":[{"x":86.65465545654297,"y":286.2382164001465,"wires":[{"id":"6e023b2.85155c4"}]}],"out":[{"x":844.5081176757812,"y":494.53173828125,"wires":[{"id":"606a1259.1bbd6c","port":2}]},{"x":841.7302913665771,"y":588.9761734008789,"wires":[{"id":"606a1259.1bbd6c","port":4},{"id":"606a1259.1bbd6c","port":5}]}],"env":[],"color":"#FFAAAA","icon":"node-red/veritone-logo-transparent.png"},{"id":"606a1259.1bbd6c","type":"switch","z":"e8111db7.09df3","name":"check task status","property":"jobStatus","propertyType":"msg","rules":[{"t":"eq","v":"queued","vt":"str"},{"t":"eq","v":"running","vt":"str"},{"t":"eq","v":"complete","vt":"str"},{"t":"eq","v":"pending","vt":"str"},{"t":"eq","v":"failed","vt":"str"},{"t":"eq","v":"aborted","vt":"str"}],"checkall":"true","repair":false,"outputs":6,"x":450,"y":480,"wires":[["ab388984.585d58"],["ab388984.585d58"],[],["ab388984.585d58"],[],[]]},{"id":"ab388984.585d58","type":"delay","z":"e8111db7.09df3","name":"Keep Polling 25s...","pauseType":"delay","timeout":"25","timeoutUnits":"seconds","rate":"1","nbRateUnits":"1","rateUnits":"second","randomFirst":"1","randomLast":"5","randomUnits":"seconds","drop":false,"x":852.1390075683594,"y":394.976318359375,"wires":[["1ca061b0.c89f7e"]]},{"id":"1ca061b0.c89f7e","type":"aiware","z":"e8111db7.09df3","name":"CheckJobStatus","format":"handlebars","syntax":"mustache","template":"query checkJobStatus{\n    job(id:\"{{jobId}}\" ) {\n    id\n    status\n    targetId\n    tasks {\n        records{\n            id\n            status\n            jobId\n            output\n            engine{\n                id\n                name\n                categoryId\n                }\n                target{\n                id\n                organization{\n                    id\n                    name\n                    }\n                }\n            }\n        }\n    }\n}","x":510,"y":280,"wires":[["ce3b92f3.78069"],[]]},{"id":"ce3b92f3.78069","type":"change","z":"e8111db7.09df3","name":"get task details","rules":[{"p":"jobStatus","t":"set","pt":"msg","to":"payload.job.status","tot":"msg"},{"p":"jobId","t":"set","pt":"msg","to":"payload.job.id","tot":"msg"},{"p":"tdoId","t":"set","pt":"msg","to":"payload.job.targetId","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":406.2858581542969,"y":373.85693359375,"wires":[["606a1259.1bbd6c"]]},{"id":"6e023b2.85155c4","type":"change","z":"e8111db7.09df3","name":"set jobId","rules":[{"p":"jobId","t":"set","pt":"msg","to":"payload.createJob.id","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":240,"y":280,"wires":[["1ca061b0.c89f7e"]]},{"id":"841b1307.fc4df","type":"subflow","name":"Transcription process","info":"","category":"","in":[{"x":60,"y":180,"wires":[{"id":"64223232.ab95dc"}]}],"out":[{"x":960,"y":120,"wires":[{"id":"4710fd05.5a5c14","port":0}]},{"x":940,"y":240,"wires":[{"id":"4710fd05.5a5c14","port":1}]}],"env":[],"color":"#FFAAAA","icon":"node-red/veritone-logo-transparent.png"},{"id":"4710fd05.5a5c14","type":"subflow:e8111db7.09df3","z":"841b1307.fc4df","name":"","env":[],"x":750,"y":180,"wires":[[],[]]},{"id":"f5417735.7adf18","type":"aiware","z":"841b1307.fc4df","name":"transcribe","format":"handlebars","syntax":"mustache","template":"mutation createJob{\n  createJob(input:{\n    isReprocessJob:true\n    targetId:\"{{tdoId}}\" #merge in payload value w/ mustache syntax\n    tasks: [\n       {\n        engineId: \"54525249-da68-4dbf-b6fe-aea9a1aefd4d\"\n        #transcription\n       }]\n  }) {\n    id\n  }\n}","x":480,"y":180,"wires":[["4710fd05.5a5c14"],[]]},{"id":"64223232.ab95dc","type":"function","z":"841b1307.fc4df","name":"Parse chunk buffer","func":"//hardcode the API url for this run\nmsg.url = 'https://api.veritone.com/v3/graphql';\n\n//Hardcoded token for Automate Training Org\nmsg.orgToken = '256235:6a4523ece2fd4cc89fbc7975b726795ce1a46d384cb1441b8ff14b22afc5f840';\n\n//Parse the string into a JSON\n//We should evaluate if this is actually a JSOn\nconst obj = JSON.parse(Buffer.from(msg.payload.aiwareChunk));\nconst {edgePayload} = obj;\n\n//Now we can start parsing the actual JSON message :-D\n//Check for tdoId value\nif(edgePayload.tdoId){\n    msg.tdoId = edgePayload.tdoId\n}\n//Now check for sendTo value\nmsg.to = edgePayload.sendTo || \"andrew+noSendTo@veritone.com\";\nmsg.firstName = edgePayload.firstName || 'AndyThere';\n//Create an event msg that saves `aiware in` payload\nmsg.event = msg.payload;\nmsg.payload.aiwareChunkObject = obj;\nreturn msg;","outputs":1,"noerr":0,"x":270,"y":180,"wires":[["f5417735.7adf18"]]},{"id":"5b3f707a.c5944","type":"subflow","name":"concat cognitive results","info":"","category":"","in":[{"x":40,"y":80,"wires":[{"id":"9c241f6.24ee7e"}]}],"out":[{"x":1120,"y":300,"wires":[{"id":"f083cdd9.2dda3","port":0}]},{"x":400,"y":200,"wires":[{"id":"9c241f6.24ee7e","port":1},{"id":"9c5dbb53.afe8d8","port":1},{"id":"dfb8420f.ecbd4","port":1}]}],"env":[],"color":"#FFAAAA","icon":"font-awesome/fa-bolt"},{"id":"3c39c314.5930dc","type":"function","z":"5b3f707a.c5944","name":"Process Results","func":"// Save TDO name to use as filename of Word document:\n\nmsg.filename = \"\";\nvar tdoNameArray = msg.payload.temporalDataObject.name.split(\".\");\n\nfor (var i = 0; i < tdoNameArray.length - 1; i++)\n{\n    msg.filename += tdoNameArray[i] + \".\";\n}\n\nmsg.filename += \"docx\";\n\n// Save Engine IDs for Transcription and Speaker Separation engine tasks that have completed on the TDO:\n\n// Engine ID of TDO's Transcription engine \nvar transcriptionEngineId = \"\";\n\n// Engine ID of TDO's translate engine\nvar translateEngineId = \"\";\n\nvar jobArray = msg.payload.temporalDataObject.jobs.records;\n\nfor (var i = 0; i < jobArray.length; i++)\n{\n\tvar taskArray = jobArray[i].tasks.records;\n\n\tfor (var j = 0; j < taskArray.length; j++)\n\t{\n\t\tvar curEngineId = taskArray[j].engine.id;\t\t\t\t\n\t\tvar curEngineCategoryId = taskArray[j].engine.category.id;\n\t\t\n\t\t// Use first Engine ID encountered for Transcription category\n\t\tif (curEngineCategoryId === \"67cd4dd0-2f75-445d-a6f0-2f297d6cd182\" && transcriptionEngineId === \"\")\n\t\t{\n\t\t\ttranscriptionEngineId = curEngineId;\n\t\t}\n\t\t// Use first Engine ID encountered for Translate category\n\t\telse if (curEngineCategoryId === \"3b2b2ff8-44aa-4db4-9b71-ff96c3bf5923\" && translateEngineId === \"\")\n\t\t{\n\t\t\ttranslateEngineId = curEngineId;\n\t\t}\n\t}\n}\n\nmsg.transcriptionEngineId = transcriptionEngineId;\nmsg.translateEngineId = translateEngineId;\n\nreturn msg;","outputs":1,"noerr":0,"x":420,"y":80,"wires":[["9c5dbb53.afe8d8"]]},{"id":"9c241f6.24ee7e","type":"aiware","z":"5b3f707a.c5944","name":"get EngineIDs","format":"handlebars","syntax":"mustache","template":"query {\n  temporalDataObject(id: \"{{tdoId}}\") {\n    name\n    jobs {\n      records {\n        id\n        createdDateTime\n        status\n        tasks(status: complete) {\n          records {\n            id\n            status\n            engine {\n              id\n              name\n              category {\n                id\n                name\n              }\n            }\n          }\n        }\n      }      \n    }\n  }\n}","x":200,"y":80,"wires":[["3c39c314.5930dc"],[]]},{"id":"9c5dbb53.afe8d8","type":"switch","z":"5b3f707a.c5944","name":"Translation Exists?","property":"translateEngineId","propertyType":"msg","rules":[{"t":"empty"},{"t":"nempty"}],"checkall":"true","repair":false,"outputs":2,"x":627.5555419921875,"y":80,"wires":[["dfb8420f.ecbd4"],[]]},{"id":"dfb8420f.ecbd4","type":"aiware","z":"5b3f707a.c5944","name":"Get Engine Results","format":"handlebars","syntax":"mustache","template":"query {\n\tengineResults(tdoId: \"{{tdoId}}\", engineIds: [\"{{transcriptionEngineId}}\"]) {\n\t  records {\n\t\ttdoId\n\t\tengineId\n\t\tstartOffsetMs\n\t\tstopOffsetMs\n\t\tjsondata\n\t\tassetId\n\t\tuserEdited\n\t  }\n\t}\n}","x":870,"y":80,"wires":[["18d0dd43.4a6b63"],[]]},{"id":"18d0dd43.4a6b63","type":"function","z":"5b3f707a.c5944","name":"format engine results","func":"\nvar transcriptionArray = msg.payload.engineResults.records[0].jsondata.series;  // JSON output of Transcription engine results\nvar textContent = \"\";\t                                                        // Content of \"text\" output\n\n// Go through each word in \"transcriptionArray\" and build \"textContent\"\nfor (var i = 0; i < transcriptionArray.length; i++)\n{\n\tvar curWord = transcriptionArray[i].words[0].word;\n\t\n\t// If current word is a period, append it to \"textContent\"\n\tif (curWord === \".\")\n\t{\n\t\ttextContent += curWord;\n\t}\n\t// Otherwise, append a space followed by the current word to \"textContent\"\n\telse\n\t{\n\t\ttextContent += \" \" + curWord;\n\t}\n}\n\nmsg.textContent = textContent.trim();\nnode.warn(\"msg.textContent = \" + msg.textContent);\n\nmsg.to = 'andrew+blmbgreport@veritone.com';\nmsg.cc = 'minh.nguyen+blmbgreport@setacinq.vn';\n\nmsg.topic = 'Your NLP Results (Flow Engine)';\n\nreturn msg;","outputs":1,"noerr":0,"x":760,"y":300,"wires":[["f083cdd9.2dda3"]]},{"id":"f083cdd9.2dda3","type":"template","z":"5b3f707a.c5944","name":"format html","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"<!DOCTYPE html PUBLIC \"-//W3C//DTD HTML 4.01 Transitional//EN\"\n   \"http://www.w3.org/TR/html4/loose.dtd\">\n<html>\n<head>\n    <meta http-equiv=\"Content-Type\" content=\"text/html; charset=UTF-8\">\n    <meta name=\"viewport\" content=\"initial-scale=1.0\">\n    <meta name=\"format-detection\" content=\"telephone=no\">\n    <title>aiWARE analytics report</title>\n    <style type=\"text/css\" data-premailer=\"ignore\">\n        .ReadMsgBody { width: 100%; background-color: #f4f4f4;}\n        .ExternalClass {width: 100%; background-color: #f4f4f4;}\n        .ExternalClass, .ExternalClass p, .ExternalClass span, .ExternalClass font, .ExternalClass td, .ExternalClass div {line-height:100%;}\n        body {-webkit-text-size-adjust:none; -ms-text-size-adjust:none;}\n        body {margin:0; padding:0;}\n        table {border-spacing:0;}\n        table td {border-collapse:collapse;}\n        .yshortcuts a {border-bottom: none !important;}\n        @media screen and (max-width: 600px), screen and (max-device-width: 600px) {\n            table[class=\"container\"] { width: 95% !important; }\n        }\n        @media screen and (max-width: 480px), screen and (max-device-width: 480px) {\n            td[class=\"container-padding\"] { padding-left: 12px !important; padding-right: 12px !important; }\n          \ttable.container td.logo { text-align: center; border-bottom: 1px solid #efefef; height:100px; color:#fff }\n        }\n    </style>\n    <style type=\"text/css\">\n        body { background-color: #f4f4f4; padding: 10px 0; margin: 10px 0; }\n        table.container { font-family: Lato, Arial, sans-serif; color: #434343; }\n        table.container td.header div { height: 10px; border: 1px solid #042f6d; }\n        table.container td.logo { background-size: cover; text-align: center; border-bottom: 1px solid #efefef; height:200px; color:#fff }\n        h1, h2, h3, h4, h5, h6 { color: #4e7ca4; line-height: 1.5; margin: 0 0 25px 0; }\n        h1 { font-size: 24px; } h2 { font-size: 18px; } h3 { font-size: 16px; } h4, h5, h6 { font-size: 14px; }\n        p { font-size: 14px; line-height: 1.5; margin: 0 0 25px 0; color: #434343; }\n        a { color: #0e4595; text-decoration: underline; }\n        td.container-padding img { border: 1px solid #ddd; padding: 37px; margin: 0 19px; }\n        a.intercom-h2b-button { background-color: #3e8acc; border-radius: 3px; color:#fff; display: inline-block; font-size: 14px; height: 44px; line-height:44px; padding: 0px 28px; text-decoration: none; }\n        ul li, ol li { font-size: 14px; line-height: 1.5; }\n        table.footer { background: #f4f4f4; width: 100%; padding-top:30px;}\n        table.footer td { vertical-align: middle; text-align: center; color: #434343; font-size: 12px; }\n      \ttable.footer p { color: #a0a0a0; font-size:12px; }\n    </style>\n</head>\n<body style=\"margin: 10px 0;padding: 10px 0;-webkit-text-size-adjust: none;-ms-text-size-adjust: none;background-color: #f4f4f4;\" leftmargin=\"0\" topmargin=\"0\" marginwidth=\"0\" marginheight=\"0\">\n<br>\n<table border=\"0\" width=\"100%\" height=\"100%\" cellpadding=\"0\" cellspacing=\"0\" bgcolor=\"#f4f4f4\" style=\"border-spacing: 0;\">\n  <tr>\n    <td align=\"center\" valign=\"top\" bgcolor=\"#f4f4f4\" style=\"background-color: #f4f4f4;border-collapse: collapse;\">\n      <table width=\"600\" cellpadding=\"0\" cellspacing=\"0\" border=\"0\" class=\"container\" bgcolor=\"#ffffff\" style=\"border-spacing: 0;font-family: Lato, Arial, sans-serif;color: #434343;\">\n        <tr>\n          <td class=\"container-padding\" bgcolor=\"#ffffff\" style=\"background-color: #ffffff;padding-left: 30px;padding-right: 30px;border-collapse: collapse;\"><br />\n            <!-- ### Begin Content! ### -->\n            <h1>Your aiWARE Analytics Report</h1>\n\n            <!-- ADD CONTENT HERE -->        \n            <p style=\"font-size: 14px; line-height: 1.5; margin: 0 0 25px 0; color: #434343;\">\n            <p>Hi {{firstName}}, here is your latest aiWARE cognitive analytics report:\n            </td>\n        </tr>\n        <tr>\n        <td valign=\"middle\" style=\"padding:10px 40px 5px; background-color:#ffffff; font-family:'Lato',Arial,sans-serif; font-size:14px;\"><table width=\"100%\">\n          <tbody>\n            <tr>\n              <td><img src=\"https://static.veritone.com/email/gear-fa.png\" width=\"18px\" style=\"height:10%;\"></td>\n              <td class=\"container-padding\" width=\"100%\" style=\"padding-left:5px;\"><span style=\"font-family:'Lato-Bold',Arial,sans-serif; font-weight:bold;\">Your Transcription Results</span>\n              </td>\n            </tr>\n          </tbody>\n        </table>\n        </td>\n        </tr>\n        <tr>\n        <td style=\"padding:5px 40px 25px 40px; background-color:#ffffff; font-family:'Lato',Arial,sans-serif; font-size:14px;\"><table width=\"100%\">\n          <tbody>\n            <tr>\n              <td style=\"padding-bottom: 5px;\">\n                <tbody>\n                  <tr>\n                    <td style=\"padding:5px 20px 5px 10px; background-color:#ffffff; font-family:'Lato',Arial,sans-serif; font-size:14px; color:#333333;\">\n                      <p style=\"margin:0px 10px 0px 20px; background-color:#ffffff; font-family:'Lato',Arial,sans-serif; font-size:14px; color#333333;\">\n                          {{{textContent}}}\n                      </p> \n                    </td>\n                  </tr>\n                </tbody>  \n              </td>\n            </tr>\n            </tbody>\n            </table></td>\n        </tr>\n        <tr>  \n        <td valign=\"middle\" style=\"padding:0px 40px; background-color:#ffffff; font-family:'Lato',Arial,sans-serif; font-size:14px;\"><table width=\"100%\">\n          <tbody>\n            <tr>\n              <td><img src=\"https://static.veritone.com/email/info-negfill.png\" width=\"18px\" style=\"height:10%;\"></td>\n              <td class=\"container-padding\" width=\"100%\" style=\"padding-left:5px;\"><span style=\"font-family:'Lato-Bold',Arial,sans-serif; font-weight:bold;\">Details</span></td>\n            </tr>\n          </tbody>\n        </table>\n        </td>\n        </tr>\n        <tr>\n        <td style=\"padding:5px 40px 25px 40px; background-color:#ffffff; font-family:'Lato',Arial,sans-serif; font-size:14px;\"><table width=\"100%\">\n            <tbody>\n              <tr>\n                <td style=\"padding: 0px 10px 0px 25px;\"><span style=\"font-family:'Lato',Arial,sans-serif;\">\n                  You can view the original file and other cognitive details for this media file by clicking the button below\n                  </td>\n                </tr>\n              </tbody>\n            </table>\n          </td>\n        </tr>\n        <tr>\n            <td class=\"container-padding\" style=\"background-color: #ffffff; padding: 10px 40px;\">\n                <center><a href=\"https://cms.veritone.com/#/media-details/{{todId}}\" style=\"background-color: #3e8acc; border-radius: 3px; color:#fff; display: inline-block; font-size: 14px; width:133px; height:44px; line-height:44px; padding: 0px 28px; text-align:center; font-family:'Lato',Arial,sans-serif; text-decoration: none;\">Media Link</a></center>\n            </td>\n        </tr>    \n        <tr>\n        <td class=\"container-padding\" bgcolor=\"#ffffff\" style=\"background-color: #ffffff;padding-left: 30px;padding-right: 30px;border-collapse: collapse;\">\n            <p style=\"font-size: 14px; line-height: 1.5; margin: 0 0 25px 0; color: #434343;\">\n            Create with care,<br>\n            Team Veritone\n            </p>\n          </td>\n        </tr>\n            <!-- ### End Content ### -->\n\n        <tr>\n            <td style=\"border-collapse: collapse;\">\n                <table border=\"0\" width=\"100%\" height=\"100%\" cellpadding=\"0\" cellspacing=\"0\" bgcolor=\"#fbfbfb\" class=\"footer\" style=\"border-spacing: 0;background: #f4f4f4;width: 100%;padding-top: 30px;\">\n                    <tr>\n                      <td style=\"border-collapse: collapse;vertical-align: middle;text-align: center;color: #434343;font-size: 12px;\">\n                        <p style=\"font-size: 12px;line-height: 1.5;margin: 0 0 25px 0;color: #a0a0a0;\">&copy; 2020 Veritone | All Rights Reserved<br>\n                        575 Anton Blvd, Costa Mesa, CA 92627\n                        </p>\n                      </td>\n                    </tr>\n                </table>\n            </td>\n        </tr>\n      </table>\n    </td>\n  </tr>\n</table>\n<br>\n</body>\n</html>","output":"str","x":970,"y":300,"wires":[[]]},{"id":"c20ef88c.0f9d88","type":"subflow","name":"error reporting","info":"","category":"","in":[],"out":[],"env":[],"color":"#FFAAAA","icon":"node-red/alert.svg"},{"id":"f469e397.cac46","type":"catch","z":"c20ef88c.0f9d88","name":"","scope":null,"uncaught":true,"x":200,"y":140,"wires":[["f369edbe.24278","3f954412.2e82dc","90c07ddc.2eb1e"]]},{"id":"f369edbe.24278","type":"debug","z":"c20ef88c.0f9d88","name":"Catch all errors","active":true,"tosidebar":true,"console":true,"tostatus":false,"complete":"payload","targetType":"msg","x":470,"y":140,"wires":[]},{"id":"2e54059b.52291a","type":"e-mail","z":"c20ef88c.0f9d88","server":"smtp.mandrillapp.com","port":"587","secure":false,"tls":false,"name":"andrew+blmbgerrs@veritone.com","dname":"email","x":650,"y":240,"wires":[]},{"id":"3f954412.2e82dc","type":"function","z":"c20ef88c.0f9d88","name":"append error objs","func":"//Structure an email subject!\nmsg.topic = `Flow Engine Runtime Error for Task - ${msg.edgeTaskId}`;\n\n//Temporary value to store the msg.payload value\nlet errPayload = msg.payload;\n\n\n//Structure a response to both Edge & Email Alert\nmsg.payload = {\n    \"Message\":\"The engine task encountered an error at runtime. See report below\",\n    \"errorBody\":msg.error,\n    \"previousPayload\":errPayload\n}\n\nreturn msg;","outputs":1,"noerr":0,"x":470,"y":240,"wires":[["2e54059b.52291a"]]},{"id":"90c07ddc.2eb1e","type":"aiware-out","z":"c20ef88c.0f9d88","name":"","statusCode":"500","x":460,"y":60,"wires":[]},{"id":"95365ac.08b9aa8","type":"tab","label":"Flow NLP Demo 1","disabled":false,"info":"# Overview\nFlow engine processing a file with Transcription and then translation\n\n## Use case\nIn this use-case, we:\n - Ingest an audio visual file from an S3 bucket\n - Write some initial metadata to the file (\"name\") and run `Transcription` processing\n - Once transcription is done, we then:\n  - run Face Recognition (detecting and recognizing) using a Library of Faces\n  - run Speaker Separation which processes the file *and* the transcript\n - Once all engine processes have finished, we then query the cognitive results,\n - We merge them all together\n - And then send out as an HTML email template to the specified recipient!"},{"id":"4ce5a841.672ff8","type":"aiware-in","z":"95365ac.08b9aa8","name":"","tdoId":"1010801274","tdoContent":"","addToJob":false,"endpointUUID":"","x":139,"y":269,"wires":[["e27b69f6.7ad228"]]},{"id":"a4010c87.172f1","type":"e-mail","z":"95365ac.08b9aa8","server":"smtp.mandrillapp.com","port":"587","secure":false,"tls":false,"name":"","dname":"email","x":909,"y":189,"wires":[]},{"id":"787edb70.c0efb4","type":"aiware-out","z":"95365ac.08b9aa8","name":"Edge OK","statusCode":200,"x":982,"y":269,"wires":[]},{"id":"c2035d02.aa3f5","type":"subflow:c20ef88c.0f9d88","z":"95365ac.08b9aa8","name":"","env":[],"x":131,"y":65,"wires":[]},{"id":"99782c6a.8d723","type":"subflow:5b3f707a.c5944","z":"95365ac.08b9aa8","name":"","env":[],"x":669,"y":249,"wires":[["a4010c87.172f1","787edb70.c0efb4"],["23c6a57c.802d7a"]]},{"id":"23c6a57c.802d7a","type":"aiware-out","z":"95365ac.08b9aa8","name":"","statusCode":"500","x":679,"y":329,"wires":[]},{"id":"e27b69f6.7ad228","type":"subflow:841b1307.fc4df","z":"95365ac.08b9aa8","name":"","env":[],"x":359,"y":269,"wires":[["99782c6a.8d723"],["23c6a57c.802d7a"]]},{"id":"33af6a9a.727ee6","type":"comment","z":"95365ac.08b9aa8","name":"Catch and Report Errors","info":"","x":155,"y":112,"wires":[]},{"id":"95f14e21.e2126","type":"comment","z":"95365ac.08b9aa8","name":"Incoming Messages from Edge","info":"","x":158,"y":343,"wires":[]},{"id":"bf408573.f663d8","type":"comment","z":"95365ac.08b9aa8","name":"Run transcription on the file","info":"This node will evaluate the value specified in the msg.payload.aiwareChunk and assuming it's a JSON that includes the `tdoId`, will process the TemporalDataObject specified","x":444,"y":199,"wires":[]}]
```

</details>

## Walkthrough

### Design the flow
Each flow starts with the aiWARE in node and
Your custom business logic
The aiWARE out node returns the message to the Edge processing framework
Deploy the flow engine in Developer App
Run the following Job to ensure that the latest engine build is available in your Edge cluster:
CreateJob { … }
Learn more about Job (link)
Send a chunk to the url defined in the Creating and Running your Engine section
Learn more about HTTP Push Adapter (link)
You will know your flow engine task completed successfully when the recipient’s email address defined in the email field receives the HTML message containing the message you sent to your custom Edge API endpoint
You can view your logs in the Edge Admin app or in Developer app if you created your engine job using GraphQL or applications like CMS or Developer
Need a page with all Jobs/tasks in your org (jobs table)


### Creating and Running your Engine Job (DAG)

#### Engine Job Setup

This is a sample Job DAG to get the creative juices flowing about how to leverage the power of Edge V3 and the flexibility of Flow Engines to extend out aiWARE with your own custom logic
URI and Header Details

Note: These details apply specifically to the Edge V3 production cluster, certain details will change if you are making an API request to another cluster

Endpoint: controller-v3f.aws-prod-rt.veritone.com/edge/v1/proc/job/create


**Headers**

```
HTTP Method: POST
“Authorization”: “Bearer <Edge API token>”
“Content-type”:”application/json”
API Body
/* Create your own endpoint using a GUID generator */
{
	"jobs": [
		{
			"dueDateTime": "2020-03-26T12:37:01.348472-07:00",
			"internalOrganizationID": "",
			"taskRoutes": [
				{
					"firstWriteDateTime": "0001-01-01T00:00:00Z",
					"lastWriteDateTime": "0001-01-01T00:00:00Z",
					"taskChildId": "PA_TASK_ID"
				},
				{
					"endpoint": "ad2e1faa-422a-4c13-a60e-291fc6980d7e",
					"firstWriteDateTime": "0001-01-01T00:00:00Z",
					"lastWriteDateTime": "0001-01-01T00:00:00Z",
					"taskChildId": "MY_ENGINE_TASK_ID",
					"taskChildInputId": "MY_INPUT",
					"taskParentId": "PA_TASK_ID",
					"taskParentOutputId": "PA_OUTPUT"
				}
			],
			"tasks": [
				{
					"correlationTaskId": "PA_TASK_ID",
					"dueDateTime": "0001-01-01T00:00:00Z",
					"engineId": "bb544ade-461c-11ea-8604-a3b3a83f5182",
					"ioFolders": [
						{
							"correlationID": "PA_OUTPUT",
							"mode": "chunk",
							"type": "output"
						}
					],
					"maxEngines": 1,
					"taskStatus": "pending"
				},
				{
					"correlationTaskId": "MY_ENGINE_TASK_ID",
					"dueDateTime": "0001-01-01T00:00:00Z",
					"engineId": "26bd9b95-9611-444e-a9e3-6036afa8d0fd",
					"ioFolders": [
						{
							"correlationID": "MY_INPUT",
							"mode": "chunk",
							"type": "input"
						}
					],
					"maxEngines": 1,
					"maxRetries": 10,
					"numChunksPerWorkItem": 1,
					"taskPayloadJSON": "{}",
					"taskStatus": "pending"
				}
			]
		}
	]
}
```

**Using your Custom Endpoint**

Request Details

You can then send your own chunks (preferably in a JSON format) to the new endpoint using the headers and resource below:

“Authorization”:”Bearer <Edge API Token>”
“Content-type”:”application/json”

https://controller-v3f.aws-prod-rt.veritone.com/edge/v1/proc/endpoint/{Your Endpoint Guid}

To ensure your engine task runs successfully, submit this JSON payload:

HTTP Method: POST
{
“edgeMsg":{
	“firstName”: “<your first name>”,
	“email”: “yourEmail@provider.com”,
	“Body”: “hello to the world from Automate Studio by Veritone!”
}
}

If you submitted your payload with the correct headers, you will receive a 200 OK response from your client (cURL, Postman, Insomnia).

The engine task is now created, pending, and will be run on the Edge framework. Assuming your payload was formatted correctly, an email will be sent to the address written as the value for the “email” field.



## FAQ

Q: How is Automate Studio-- and the flow engine-- different from vanilla Node-RED?
A: Think of Automate Studio and the Node-RED editor as your authoring and debugging tool. You can create and run your flow in Automate Studio, but flow engine runs against production data as a headless flow engine that can run multi-tenant in any Veritone organization that has access to that engine

Q: If I create an engine job on aiWARE Edge, can I see the flow processing the data in Automate Studio?
A: This feature is not currently supported. Think of Automate Studio and Edge as two separate runtime environments

Q: What is the difference between using GraphQL to create an engine job vs using Edge API?
A: Think of GraphQL as providing another layer between you and the engine processing framework. There are reasons and tradeoffs for using either approach, the Edge API is useful for connecting directly to a specific 

## Implementation notes

- Earlier versions are using a `msg.orgToken` variable to ensure that the flow engine has access to the createJob mutation to run the engine.
- This is **not** recommended especially for making your flows available across multiple orgs
- Instead, the `jwtRights` field will be automatically written with the rights designated for your flow engine


## Recommended Reading

Our mission with Automate Studio is to create an intuitive process for processing media. We are building towards this mission and know we have a lot to learn along the way!

In the meantime, here are some recommended aiWARE Docs to familiarize yourself with the deeper points of aiWARE, running engines, and processing:

- [Overview of Veritone GraphQL API](https://docs.veritone.com/#/apis/using-graphql)
- [Engine Job Quickstart Guide](https://docs.veritone.com/#/apis/job-quickstart/)
- [More Reading on Jobs, Tasks, and TDOs](https://docs.veritone.com/#/apis/jobs-tasks-tdos)
- [Creating and Processing a Temporal Data Object (TDO)](https://docs.veritone.com/#/apis/tutorials/upload-and-process)
- [Working with the Edge REST API](https://docs.veritone.com/#/apis/edge/index.html)
