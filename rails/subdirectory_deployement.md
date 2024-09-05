# Deployment of Rails Application in a Nginx sub-directory

> Here, we will deploy our application which is an admin panel in the */admin_panel* sub-directory  
>    
> ubuntu 22.04  
> ruby 3.3.0  
> Rails 7.1.3.2

## Steps
1. Change Nginx config
2. Change in rails config

### Change Nginx Config
Steps:  
1. Go to "/etc/nginx/sites-available/default"
   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```
2. For sub-directory deployment the location needs to be changed. Add the following locations for rails application.
   ```bash
    location /admin_panel {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /admin_panel/cable {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade "websocket";
          proxy_set_header Connection "Upgrade";
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
   ```
> location */admin_panel* is to serve the main rails application  
> location */admin_panel/cable* for action cable or web-socket connections  

3. Test and reload nginx with new config
   ```bash
   sudo nginx -t
   sudo nginx -s reload
   ```
> If there is any error get help from google or chatgpt

### Change in rails config
Steps:
1. Change **config.ru** file
  ```bash
    map '/admin_panel' do
      run Rails.application
    end
  ```
  > The file already has ```run Rails.application``` line. Surround the line with map.

2. Change **config/application.rb**
  ```bash
    config.active_storage.routes_prefix = '/admin_panel'
  ```
  > Add the given line just after ```class Application < Rails::Application``` line. It will be the first line inside the class.

3. Change environment files **config/environments/<environment_name>.rb**
  ```bash
    config.relative_url_root = "/admin_panel"
    config.action_controller.relative_url_root = "/admin_panel"
    ENV['RAILS_RELATIVE_URL_ROOT']  = "/admin_panel"
    ENV['ROOT_URL']  = "/admin_panel"
  ```
  > Environment can be *development*, *staging*, *production* etc. Change the environment as you require.  
  > Add the given lines inside the rails application configuration just after ```Rails.application.configure do``` line.
