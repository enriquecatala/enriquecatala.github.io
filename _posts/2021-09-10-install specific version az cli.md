---
layout: post
title:  "How to use a specific version of az cli"
date:   2021-09-10 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: DataNinja WSL2 OS Azure
---

There are some times when the latest version of az cli is not working. Like for example today while I was trying to debug a container issue running in Azure Container Instances.

I´m getting the following error while trying to connect and check why my container is not working:
```bash
~#@❯ az container logs --resource-group "InteligenciaAlertasSentinel" --name detector
The command failed with an unexpected error. Here is the traceback:
'ContainerInstanceManagementClient' object has no attribute 'container'
Traceback (most recent call last):
  File "/opt/az/lib/python3.6/site-packages/knack/cli.py", line 231, in invoke
    cmd_result = self.invocation.execute(args)
  File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 657, in execute
    raise ex
  File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 720, in _run_jobs_serially
    results.append(self._run_job(expanded_arg, cmd_copy))
  File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 691, in _run_job
    result = cmd_copy(params)
  File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 328, in __call__
    return self.handler(*args, **kwargs)
  File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/command_operation.py", line 112, in handler
    client = self.client_factory(self.cli_ctx, command_args) if self.client_factory else None
  File "/opt/az/lib/python3.6/site-packages/azure/cli/command_modules/container/_client_factory.py", line 18, in cf_container
    return _container_instance_client_factory(cli_ctx).container
AttributeError: 'ContainerInstanceManagementClient' object has no attribute 'container'
To open an issue, please run: 'az feedback'
```

It seems the problem is [related to the version of az cli](https://github.com/Azure/azure-cli/issues/19475) that I´m using (the last one while typing this post).

For situations like this, i recommend you to use the [official docker repo of azure az cli](https://hub.docker.com/_/microsoft-azure-cli) which easily allows me to use a [specific version of az cli](https://mcrflowprodcentralus.data.mcr.microsoft.com/mcrprod/azure-cli?P1=1631287029&P2=1&P3=1&P4=4G2Xm%2FZgSoKLX2W856%2Feoxty3El5gyBaY2xnTS2KpLQ%3D&se=2021-09-10T15%3A17%3A09Z&sig=TDg2ib8Q5JU5FkGlx%2B7YQTXVQCJyPB8jPV6kIPf%2FEcc%3D&sp=r&sr=b&sv=2015-02-21).


<!--end_excerpt-->

If you have docker installed on your machine you can install the latest version of az cli using the following command:

```bash
docker run -it mcr.microsoft.com/azure-cli:<version>
```
>NOTE: last version working is 2.27.2

And that´s it!, as easy as that :)