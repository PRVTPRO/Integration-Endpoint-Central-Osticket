# Endpoint Central + osTicket 
## Endpoint Central (The token can be obtained in different ways see the API Manual)

Request post
`https://SERVER_EndpointCentral/api/1.4/desktop/authentication`
```
        {
             "username":"USERNAME",
             "password":"PASSWORD IN BASE64",
             "auth_type":"ad_authentication" ,
             "domainName": "domain.local",
        }
```
Answer
```
{
    "message_type": "authentication",
    "message_response": {
        "authentication": {
            "two_factor_data": {
                "unique_userID": "USERNAME",
                "google_authenticator_key": "LELSY21KE3UREO",
                "is_TwoFactor_Enabled": true,
                "remember_token_days": 120,
                "OTP_Validation_Required": true
            }
        }
    },
    "message_version": "1.4",
    "status": "success"
}
```
We are interested in the parameter unique_user ID

Request post
`https://SERVER_EndpointCentral/api/1.4/desktop/authentication/otpValidate`
```
{
 "uid":"HERE UID",
 "otp":"CODE OTP",
 "rememberme_enabled":"true"
}
```
In the answer we find device_token example ` '"device_token": "$2a$12$0j5b5.nhkiopiaa.tzlzlzesnov0omzr4oz7dotzhp5fos"`


Post request with added device token
`https://SERVER_EndpointCentral/api/1.4/desktop/authentication`
```
        {
             "username":"USER NAME",
             "password":"PASSWORD IN BASE64",
             "auth_type":"ad_authentication" ,
             "domainName": "domain.local",
             "device_token":"$2a$12$0J5W5Kovi.nhQuIOpIaa.tZlzLzEsnOv0OmzR4OZ7DoqjP5FoS"
        }
```
In the response, we find the auth_token parameter, this is our authorization token, the validity period is set to 120 days, following from "remember_token_days": 120
```
            "auth_data": {
                "auth_token": "E19A3E0-7F0C-4EEB-8E92-7C62D2FC2"
            }
```
## osTicket

`Opening include/staff/ticket-view.inc.php ~ 264 line after it we add `

```
                <tr>
                    <th><?php echo __('PC'); ?>:</th>
                    <td><?php
                    
                        $str=strpos($ticket->getEmail(), "@");
                        $row=substr($ticket->getEmail(), 0, $str);

                        $url = 'https://SERVER_EndpointCentral/api/1.4/som/computers?searchtype=agent_logged_on_users&searchcolumn=agent_logged_on_users&searchvalue='.$row;

                        $data = array();

                        $options = array(
                            'http' => array(
                                'header'  => "Authorization: ENTER THE RECEIVED TOKEN HERE!!!\r\n",
                                'method'  => 'GET',
                                'content' => http_build_query($data)
                            )
                        );
                        $context  = stream_context_create($options);
                        $result = file_get_contents($url, false, $context);
                        if ($result === FALSE) { /* Handle error */ }

                        $json_a = json_decode($result, true);
                        echo $json_a['error_description'];
                        if ($json_a['message_response'][total] < 1){
                            echo ' Device not found';
                        }
                        else
                        {
                            foreach($json_a['message_response'][computers] as $item) {
                                echo '🖥️ <a href="https://SERVER_EndpointCentral/webclient#/uems/inventory/computers/'. $item['resource_id'] .'" target="_blank">' . $item['full_name'] . '</a>/' . $item['ip_address'] . '<br />';
                            }

                        }
                        //var_dump($result);
                        
                        ?>
                    </td>
                </tr>
```







