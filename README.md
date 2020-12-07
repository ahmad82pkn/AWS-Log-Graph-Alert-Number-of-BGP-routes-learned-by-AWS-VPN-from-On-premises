# AWS-Log-Graph-Alert-Number-of-BGP-routes-learned-by-AWS-VPN-from-On-premises
This Lambda Function will push BGP route count on your AWS VPN to Cloudwatch Logs to further Graph or Alert when certain threshold is breached

Pre Req: 
Create a Log Group and Log stream in Cloudwatch Logs. I am using below in the code , change as per your naming preferences.

logGroupName='BGPLOGS',logStreamNamePrefix='VpnStream



Make sure , Role associated with Lambda has permission to log to cloudwatch logs.



You can add trigger with cloudwatch event cron expression to run it every minute or 5 minutes, up to you.

Further you can Create metric Filter per VPN ID on your log group and alert if number of routes learned/received at AWS side increase of decrease from certrain threshold.

Sample Metric filter pattern 1
Filter pattern[number=* , vpn=vpn-0xxxx, PeerIP=13.237.x.x]

Sample Metric filter pattern 2
Filter pattern [number=*,vpn=vpn-0d8b08736b7a62ebd]


----------------------------------------------

Code: Feel free to change variable names to something more meaningful as per your needs


import json
import boto3
import time

def lambda_handler(event, context):
    # TODO implement
    millis=int(round(time.time()*1000))



    myec2Sydney = boto3.client('ec2',region_name='ap-southeast-2')
    clogs= boto3.client('logs',region_name='ap-southeast-2')




    responseSydney=myec2Sydney.describe_vpn_connections(
    )
    for eachVpn in responseSydney['VpnConnections']:
        for eachTunnelOfVpn in eachVpn['VgwTelemetry']:
                AcceptedRouteCountSydney=str(eachTunnelOfVpn['AcceptedRouteCount'])
                VpnConnectionIdSydney=str(eachVpn['VpnConnectionId'])
                OutsideIPAddress=str(eachTunnelOfVpn['OutsideIpAddress'])
                if eachTunnelOfVpn['Status']=="DOWN":
                    AcceptedRouteCountSydney="0"
                fullmessageSydney=AcceptedRouteCountSydney+" "+VpnConnectionIdSydney+" "+OutsideIPAddress
                gettokenresponse=clogs.describe_log_streams(logGroupName='BGPLOGS',logStreamNamePrefix='VpnStream')
                if "uploadSequenceToken" in gettokenresponse['logStreams'][0]:
                        nexttoken=str(gettokenresponse['logStreams'][0]['uploadSequenceToken'])

                        cresponse=clogs.put_log_events(
                        logGroupName='BGPLOGS',
                        logStreamName='VpnStream',
                        logEvents=[
                        {
                                'timestamp':millis,
                                'message':fullmessageSydney
                                        },
                        ],
                        sequenceToken=nexttoken
                        )
                else:
                        cresponse=clogs.put_log_events(
                        logGroupName='BGPLOGS',
                        logStreamName='VpnStream',
                        logEvents=[
                        {
                                'timestamp':millis,
                                'message':fullmessageSydney
                                        },
                        ],
                        )
