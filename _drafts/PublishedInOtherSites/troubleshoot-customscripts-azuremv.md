---
layout: post
title:  "Troubleshoot CustomScripts in Azure"
date:   2021-12-02 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: Dev OS Azure
---

Los [CustomScript](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-linux) components de Azure son una forma muy interesante de ejecutar código durante el despliegue de una nueva VM. Son muy útiles por ejemplo para aprovisionar una VM con nuestras configuraciones/software preferido y tenerlas listas para cuando finalice el despliegue.

En mi caso concreto, yo lo uso para configurar ciertas partes de HW, como activar HW cuda, instalar visual studio, [dvc](https://dvc.org/) y ciertas librerias optimizadas que necesito para trabajar en el campo del deeplearning. 

Es importante conocer que cuando algo pasa, siempre tienes acceso a la ruta por defecto donde se encuentra tanto el log de ejecución, como el log de errores así como toda carpeta/fichero que tu script haya descargado/creado.

## Añadir CustomScript con terraform

La forma mas facil de hacerlo es utilizando el recurso [azurerm_virtual_machine_extension](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_machine_extension): 

```terraform
resource "azurerm_virtual_machine_extension" "post_deployment" {
  name                 = "${var.vm_name}-vmext"
  virtual_machine_id   = azurerm_linux_virtual_machine.vm.id
  publisher            = "Microsoft.Azure.Extensions"
  type                 = "CustomScript"
  type_handler_version = "2.0"

  protected_settings = <<PROT
    {
        "script": "${base64encode(file(var.post_deployment_script))}"
    }
  PROT

  tags = var.tags
}
```

En este caso, en el contenido de la variable _var.post_deployment_script_ lo que hay es la ruta local al fichero donde está mi script, que como ves se envía como **base64** en la propiedad _protected_settings_ del componente.

Esto básicamente lo que hace es que cuando la máquina sea desplegada, ejecutará el código que tengas ahi dentro durante su despliegue, tan simple y efectivo como eso. En mi caso es un script como este:

```bash
#!/bin/bash

echo "**********************************"
echo "Activating CUDA Hardware..."
bash /tmp/activate_cuda.sh 
echo "**********************************"
echo "Configure python..."
bash /tmp/configure_python.sh 
echo "**********************************"
echo "Configuring software..."
bash /tmp/configure_software.sh 
```
>NOTA: El script hace referencia a otros scripts, que tienen que estar subidos previamente, para lo que puedes utilizar  provisioner "file" de terraform 

El script anterior, en su primer elemento descarga una serie de librerias muy específicas que debo ejecutar en orden y que de no hacerlo, no tendré activas ciertas optimizaciones hardware de la máquina sobre la que lo despliego.

# Problema a resolver

En mi caso, me di cuenta que despues de desplegar la máquina, tanto _configure_python.sh_ como _configure_software.sh_ ejecutaron correctamente, pero por alguna razón, las máquinas no podian hacer uso de hardware CUDA, como si el primer script llamado activate_cuda.sh hubiera fallado en alguna parte.

## Troubleshoot

Pues bien, lo que hay que hacer en ese caso es entrar a la carpeta donde se vuelca en fichero y que contiene toda traza de su ejecución, incluyendo _stdout_ y _stderr_ y que **siempre va a estar aqui**:

```bash
cd /var/lib/waagent/custom-script/download/0/
```

En esta carpeta encontrarás no solo el log de ejecución, sino además todo lo que se haya descargado/creado durante la ejecución del script.

```bash
(base) root@datascience:/var/lib/waagent/custom-script/download/0# ls
cuda-repo-ubuntu1804-11-4-local_11.4.0-470.42.01-1_amd64.deb  libnvinfer-dev_8.2.1-1+cuda11.4_amd64.deb         libnvinfer8_8.2.1-1+cuda11.4_amd64.deb  stdout
libcudnn8-dev_8.2.4.15-1+cuda11.4_amd64.deb                   libnvinfer-plugin-dev_8.2.1-1+cuda11.4_amd64.deb  script.sh
libcudnn8_8.2.4.15-1+cuda11.4_amd64.deb                       libnvinfer-plugin8_8.2.1-1+cuda11.4_amd64.deb     stderr
(base) root@datascience:/var/lib/waagent/custom-script/download/0# 
```

Y ahora solo te quedaría revisar por tanto que puedes sacar del contenido de stderr, por ejemplo.

En mi caso, lo que vi fue esto:

```bash
2021-12-01 21:43:41 (186 MB/s) - ‘cuda-repo-ubuntu1804-11-4-local_11.4.0-470.42.01-1_amd64.deb’ saved [2427635668/2427635668]

dpkg: error: dpkg frontend is locked by another process
Warning: apt-key output should not be parsed (stdout is not a terminal)
```

Que era básicamente un bug en mi script de terraform, porque olvidé incluir la directiva de dependencia que evitase ejecutar varios _azurerm_virtual_machine_extension_ que tengo desplegando en esa máquina.

La solución era facil esta vez y consistía simplemente en:

```terraform
resource "azurerm_virtual_machine_extension" "post_deployment" {
  depends_on = [
     azurerm_linux_virtual_machine.vm,
     azurerm_virtual_machine_extension.monitoring
  ]
  ...
}
```

Una vez hecho esto, problema resuelto :)