+++
date = '2026-07-11T21:15:39+02:00'
draft = false
title = 'First'
+++
# The Blog Build
---
The first project is this blog. 
The purpose is to have somewhere to easily document projects.
Since I want to expand my knowledge of, and get experience with, Azure DevOps this was a great opportunity to do just that. 
## Technologies Used
- [Hugo](https://gohugo.io) for generating the static website.
- [GitHub](https://github.com) for source code management.
- [Azure DevOps](https://azure.microsoft.com/en-us/products/devops) for automating build and deployment pipeline.
- [Azure Static Web Apps](https://azure.microsoft.com/en-us/products/app-service/static) for hosting and automated SSL certificate management.
- [Azure DNS](https://azure.microsoft.com/en-us/products/dns) for custom domain name resolution and routing.

## Setting it all up

Installing Hugo and implementing a template was very straight forward when following the documentation for Hugo.
Theme, amazing as it is, didn't really have favicons. In order to implement my own awesome one, I had to copy the header file to the `layouts/partials` folder and add a line: 
```HTML
<link rel="icon" type="image/png" href="/Images/favicon.png">
```
That way Hugo will prioritize my header file rather than the one in `themes/loficode/layouts/paritals`.
There is just one problem. If the creator of the theme updates this file it could break the site, or the build process. Since Azure DevOps will pull the theme straight from the creator's GitHub when automatically building the site, the pipeline will always fetch the latest version of the theme. Because Hugo prioritzes my header file it will not get any updates automatically.

## Getting the YAML Pipeline to work
There was some issues with getting it to build and deploy as it should.
Firstly Microsoft Oryx didn't want to build with the latest version of `Hugo v0.164.0`. So I tried to specify a version that Microsoft Oryx likes to work with, `Hugo v0.148.2`. 
However, now the theme didn't want to work with this older version of Hugo. 
The solution was to write a small script that installs `Hugo v0.164.0` directly on to the pipeline agent, and compiles the HTML. Then the Azure task will just have to function as an uploader. 
```YAML
steps:
  - checkout: self
    submodules: true

  # Downloads hugo v0.164.0, installs it, and compiles the HTML natively
  - script: |
      curl -L -o hugo.deb https://github.com/gohugoio/hugo/releases/download/v0.164.0/hugo_extended_0.164.0_linux-amd64.deb
      sudo dpkg -i hugo.deb
      hugo --minify


  # Upload the pre-built public folder
  - task: AzureStaticWebApp@0
    inputs:
      app_location: '/' 
      output_location: 'public'
      skip_app_build: true # tells Oryx to step back and just upload. 
      azure_static_web_apps_api_token: $(deployment_token)
```

There were some small issues here and there but nothing very interesting to go through here.

