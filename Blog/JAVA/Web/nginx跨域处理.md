```
    #api

    location ^~ /api{

        add_header Access-Control-Allow-Credentials true ;
        add_header 'Access-Control-Allow-Origin' *;
        #add_header Access-Control-Allow-Origin 'ip:port'
        #add_header Access-Control-Allow-Methods 'POST,GET,OPTIONS,PUT,DELETE' always;
        add_header Access-Control-Allow-Methods *;
        add_header Access-Control-Allow-Headers *;
        add_header Access-Control-Expose-Headers 'Content-Disposition,WWW-Authenticate,Server-Authorization' always;
        # OPTIONS类的请求，是跨域先验请求
        if ($request_method = 'OPTIONS') {
               return 204; # http状态码 204 （无内容） 服务器成功处理了请求，但没有返回任何内容。可以返回 200   
        }
        #proxy_hide_header
        proxy_pass http://apigateway_server;
        proxy_redirect http:// https://;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_hide_header Access-Control-Allow-Origin;
        proxy_hide_header Access-Control-Allow-Credentials;  
        send_timeout 30s;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;
        proxy_connect_timeout 1s;

    }
```