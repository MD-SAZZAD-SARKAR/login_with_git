## Config File   config.php
```php
<?php
// Start session 
if (!session_id()) {
    session_start();
}

class github
{
    // your client_id and client_secret
    public $client_id = 'Iv1.9d65f06c79d36c23';
    public $client_secret = 'ed867a571dbab445c96fd862f7fb5edc49058013';

    public $authorizeURL = "https://github.com/login/oauth/authorize";
    public $tokenURL = "https://github.com/login/oauth/access_token";
    public $apiURLBase = "https://api.github.com";



    // get user code
    function usercode()
    {
        // method one
        // header('location:'.$this->authorizeURL.'?client_id='.$this->client_id);

        // another method use array
        $param = ['client_id' => $this->client_id];
        header('location:' . $this->authorizeURL . '?' . http_build_query($param));
    }


    // get token user
    function usertoken()
    {
        $git_code = $_SESSION['git_code'];
        $pram = [
            'client_id' => $this->client_id,
            'client_secret' => $this->client_secret,
            'code' => $git_code
        ];

        $context = stream_context_create(
            [
                "http" => [
                    'method' => 'GET',
                    'user_agent' => 'Login-Github',
                    'header' => 'Accept: application/json'
                ]
            ]
        );

        $access_token_url = $this->tokenURL . '?' . http_build_query($pram);
        $response = file_get_contents($access_token_url, false, $context);
        return $response ? json_decode($response, true) : $response;
    }



    // get user info
    function getUserInfo($access_token)
    {
        $apiURL = $this->apiURLBase . '/user';

        $ch = curl_init();
        curl_setopt_array(
            $ch,
            [
                CURLOPT_URL => $apiURL,
                CURLOPT_HTTPHEADER => array('Content-Type: application/json', 'Authorization: token ' . $access_token),
                CURLOPT_RETURNTRANSFER => 1,
                CURLOPT_USERAGENT => 'login-github',
                CURLOPT_SSL_VERIFYPEER => false,
                CURLOPT_CUSTOMREQUEST => "GET"
            ]
        );

        $api_response = curl_exec($ch);

        $http_code = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        if ($http_code != 200) {
            if (curl_errno($ch)) {
                $error_msg = curl_error(($ch));
            } else {
                $error_msg = $api_response;
            }
            throw new Exception(("Error " . $http_code . ": " . $error_msg));
        } else {
            return json_decode($api_response, true);
        }
    }
}

```


## index file index.php

```php
<?php
require 'config.php';
$git = new github();

// assign athenticated user code
if (isset($_GET['code'])) {
    $_SESSION['git_code'] = $_GET['code'];
}
// end assign athenticated user code

if (isset($_SESSION['git_user'])) {
    // ============================= write your code ======================



    echo '<pre>';
    print_r($_SESSION['git_user']);
    echo '</pre>';



    //============================== end write your code =============================
} else if (isset($_SESSION['git_token'])) {
    // get user info
    $git_token = $_SESSION['git_token']['access_token'];
    $user_data = $git->getUserInfo($git_token);
    $_SESSION['git_user'] = $user_data;
    header('location:./');
    // end get user info
} else if (isset($_SESSION['git_code'])) {
    // get token
    $token =   $git->usertoken();
    $_SESSION['git_token'] = $token;
    header('location:./');
    // end get token
} else {
    // get code
    $git->usercode();
    // end get code
}

```