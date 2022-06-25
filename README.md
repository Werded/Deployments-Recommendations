# Deployments-Recommendations
Recommendations for deployments with Kubernetes 


https://github.com/bitnami/charts - чарты битнами( топовая компания)

Установка существующих чартов в namespase
1. helm install {chart name} bitnami/<chart> -f values.yaml(кастомные вэлью чарта) -n {namepspace}
2. helm list -n {namepspace} - проверяем установился ли чарт.
3. helm delete {chart name} -n {namespace}


Установка чарта проекта:

1. kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>(create registry credentials for kubernetes, without this credentials k8s cannot pull image of app from your registry).

2. kubectl apply -f pv-claim.yaml(настройки для pvc) -n {namepspace} - создание PVC для вашего чарта

3. kubectl get pvс -n {namepspace} - проверяем установился ли pvc

4. Идем в helm_vars выбираем нужное окружение и в values.yaml в строке persisntace добавляем новую переменную existingClaim={name of created pvc}. Для каждого окружения отдельный pvc.

5. helm secrets install {chart name} {path to custom chart} -f {env}/values.yaml(вэлью из helm_vars) -f {env}secrets.yaml(секреты из helm_vars) -n {namepspace} --set image.tag={image tag in docker registry(usually its giltab project registry)}





Cоздание секрета из файла:

1. kubectl create secret {name of secret} --from-file={path to file} -n {namespace}



Поднять Https: 
В кубере Https сертификат генерируется автоматом.
Нужно задать в helm_vars values:
ingress:
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: {your host}
      paths: [/]
  tls:
    - secretName: {your secret name}
      hosts:
        - {your host}


Зайти в постгрес:

kubectl exec -it {pod name} --namespace={namespace} -- psql -U {user}

Удалить PVC:

kubectl delete pvc {pvc name} --namespace={namespace}



Все релизы в хелм:
helm list -n {namespace} -a




Создание кластера в Google Cloud:
1. Создаем всё через интерфейс(выбираем настройки и мощности)
2. Подключаем к kubectl:
	1. Устанавливаем gcloud : snap install google-cloud-sdk --classic
	2. gcloud init
	3. Генерим конфиг для kubectl gcloud container clusters get-credentials {cluster name}  --region={region name}
	
	
Создание сертификата:
1. Cоздаем namespace cert-manager
2. Устанавливаем чарт:
	helm repo add jetstack https://charts.jetstack.io
	helm repo update
	helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.0.2 --set installCRDs=true
2. Создаем Issuer: kubectl apply -f cluster-issuer.yaml -n cert-manager


DUMP BD 
kubectl exec -it stg-db-postgresql-0 -n {namespace} -- bash -c "pg_dump -U {user} {database_name}" > database.sql

GET indexes of TABLE
\d { table_name};

DROP index
DROP INDEX {index_name};

Change column type in POSTGRES
The USING clause specifies an expression that allows you to convert the old values to the new ones
ALTER TABLE {table_name} ALTER COLUMN {column_name} TYPE {type} USING {column_name}::{type};
	
GKE Private cluster Authorized Networks: https://www.youtube.com/watch?v=wlnrXBodkB4&ab_channel=CloudAdvocate (how to connect to private cluster)
