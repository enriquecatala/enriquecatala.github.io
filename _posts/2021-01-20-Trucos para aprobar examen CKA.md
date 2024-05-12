---
layout: post
title:  "10 trucos para aprobar el examen CKA"
date:   2021-01-20 00:00:00 +0200
tipue_search_active: true
comments: true
excerpt_separator: <!--end_excerpt-->
tags: Exam Kubernetes DataNinja
---

Hoy mismo recibo el mail indicándome que acabo de aprobar el examen [CKA: Certified Kubernetes Administrator](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/). El examen es bastante chulo porque se aparta del típico examen al que estaba acostumbrado (al menos yo) de tipo test. En este caso, se miden realmente tus capacidades resolutivas con la tecnologia, porque el examen se trata de 17 preguntas, de diferente dificultad, donde te dan un objetivo, una linea de comandos y un tiempo para conseguirlo. La verdad que he disfrutado muchísimo y al margen de lo dificil/facil que te pueda parecer, la experiencia es bastante satisfactoria.

En este post te voy a contar las cosas que a mi modo de ver debes tener bien presentes antes de embarcarte en la certificacion CKA

<!--end_excerpt-->

![cka exam Enrique Catalá](/img/posts/cka-exam-passed/cka-enrique-catala-bañuls.png)

# Hazte experto en kubectl

> <mark>¡EL EXAMEN NO DA TIEMPO A FINALIZARLO SI NO CONOCES KUBECTL!</mark>

Debes dominar al extremo el comando kubectl porque el tiempo es realmente escaso y debes maximizar y priorizar tus opciones al máximo. 

Te recomiendo que al menos controles fuertemente comandos como los siguientes, porque **serán la diferencia entre aprobar o suspender por el tiempo de montar a mano yaml**

```bash
# explain commands
kubectl explain pod --recursive | grep -A5 tolerations

# create pod yaml
kubectl run pod nginx \
			--image=nginx \
			--labels=tier=db \
			--dry-run=client \
			--namespace=myns \
			-o yaml > file.yaml

# create deployment yaml
kubectl create deployment nginx  \
					--image=nginx \
					--dry-run=client \
					--namespace=myns \
					-o yaml > file.yaml

# create service yaml
kubectl create service clusterip myservice \
                    --tcp=80:80 \
                    --dry-run=client  \
                    --namespace=myns \
                    -o yaml > file.yaml

# change namespace
kubectl config set-context $(kubectl config current-context) --namespace=dev

# exec something from inside a pod
kubectl exec -it mypod -- nslookup mysql.namespace.svc.cluster.local

# create configmaps
kubectl create configmap nginx-configuration --namespace=ingress-space --dry-run=client -o yaml > nginx-configuration.yaml

# create serviceaccounts
kubectl create serviceaccount ingress-serviceaccount --namespace=ingress-space --dry-run=client -o yaml > ingress-serviceaccount.yaml

```

# Controla jsonpath

En mas de una pregunta vas a tener que usar indirectamente busquedas con jsonpath, porque te pediran hacer tal o cual cosa contra ciertos objetos, que deberas encontrar entre una lista bastante importante. Recuerda hacerte amigo de:

- kubectl ... --sort-by==
- kubectl ... -o jsonpath=''
- kubectl ... -o=custom-columns=''

# Conoce bien cómo interactuan los servicios internamente

Hazte al ánimo que al menos una pregunta pesada (de 15 puntos) va a ser de reparar un cluster con un worker node caido. Entiende bien cómo funcionan los servicios con [journalctl -u](https://www.freedesktop.org/software/systemd/man/journalctl.html), [systemctl](https://www.freedesktop.org/software/systemd/man/systemctl.html), conoce cómo despliega kubernetes los pods estáticos (típicamente en /etc/kubernetes/manifest/, pero igual en tu pregunta no están ahi...just saying :)). En fin, que entiendas bien cómo se levantan los servicios, qué certificados, claves y CA utilizan...de dónde salen las configuraciones de kubelet,...

Para aprenderlo bien, te recomiendo que te instales como mínimo una vez un [cluster con kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/) y un cluster [usando la version dura](https://github.com/kelseyhightower/kubernetes-the-hard-way) y que te centres en entender cómo va cada cosa mas allá de hacer copy-paste para que te sirva en troubleshooting futuros.

# Créate bookmarks en chrome

Durante el examen permiten una pestaña de chrome con acceso completo a https://kubernetes.io/docs/home y aunque tienes un buen buscador...siempre hay cosas que cuando estudias, seguro que se te quedan algo cojas y conviene tenerlas en el bookmark por si te preguntan sobre ello. 
Como mínimo te recomiendo que tengas preparadas las siguientes:
- [CheatSheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Install with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#steps-for-the-first-control-plane-node)
- [DNS debug](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)
- [NGINX ingress deployment](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)

A partir de ahi, las que tu consideres pero piensa que si te pasas poniendote marcadores...igual acabas teniendo que usar el buscador :)

Recuerda que las preguntas no van a ser directamente preguntarte cosas de esas webs, esas webs son buena referencia para cosas que por ejemplo no recuerdes (en mi caso por poner un ejemplo a la hora de crear un usuario con certificado, siempre se me olvida cómo generar el base64 del certificado para incrustarlo en el yaml...)

# Destina tus 2 primeros minutos a configurar tu entorno

Recuerda, cada segundo cuenta, destina 2 minutos a configurar tus atajos de teclado

```bash
source <(kubectl completion bash) 
echo "source <(kubectl completion bash)" >> ~/.bashrc 
alias k=kubectl
complete -F __start_kubectl k
```

# Revisa el namespace y el cluster SIEMPRE!

Cada pregunta tiene al principio un comando kubectl que debes lanzar para saltar al cluster kubernetes donde resolver la pregunta en cuestión. Recuerda hacerlo nada mas cambiar de pregunta.
Hay preguntas donde te van a decir que hagas tal o cual cosa con objetos dobre diferentes namespace. Lee bien la pregunta porque si lo haces en el namespace que no sea...la pregunta es 0 :)

# Usa tu teclado y ratón

Me arrepentiré siempre de no haber usado mi teclado y haber ido al examen con el laptop y su tedioso teclado. Enserio, acabé la última pregunta a 1m de finalizar el examen por la cantidad de errores que estuve metiendo constantemente al escribir comandos en un teclado que no era el habitual. Recuerda, **cada segundo cuenta**

# Editor de texto vi

Aprende vi. Si no sabes usar vi, aprende vi. ¿te he dicho ya que aprendas vi? pues eso :)

# Hazte buscador experto en kubernetes.io

Pues eso, van a salirte cosas muy variopintas, mas te vale conocer muy a fondo la estructura de la web kubernetes.io, porque seguro que la consultas mas de lo que piensas y acabar perdido entre su "peculiar" forma de organizar información no debe ser nada agradable cuando el tiempo vuela :)

# Olvídate de buscar atajos con supuestas empresas que te dan preguntas/respuestas

Si eres de esos que se compran packs de preguntas de examen, te las estudias de memoria y se presentan...este examen no es para ti. En general me da mucha rabia la gente que hace ese tipo de cosas, pero eso ya es otro tema de discusión aqui. Lo que quiero que entiendas es que realmente este examen va sobre tener las cosas claras y de hecho te dejan total libertad para abrirte una pestaña de navegador y acceder a las siguientes URL:
- https://kubernetes.io/docs/home/
- https://github.com/kubernetes

En este examen no hay un tipo de pregunta tipo test y cada pregunta parece random y con cositas diferentes que no tiene ningun sentido saberse de memoria teniendo acceso completo a los recursos de la propia web kubernetes para consultarlos

# Suerte!

Si estas leyendo este post entiendo que es porque tienes pensado presentarte, por lo que te deseo que lo consigas pronto!