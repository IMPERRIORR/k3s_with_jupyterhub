# Инструкция по уставновке k3s и jupterhub.
![alt-текст](https://avatars.mds.yandex.net/i?id=be872ffb6c448dc5d6b5f67ed2a9f80b-4805918-images-thumbs&n=13 "Текст заголовка логотипа 1")


В данной инструкции подробно написанно, как развернуть на вашем сервере jupyterhub с подключенным к нему системой автоматизации развёртывания и масштабирования контейнеризированных приложений k3s.
## Уставновка k3s на ваш сервер.
Измени hostname на нодах

`sudo hostnamectl set-hostname srvk3s-master-1`


Инициализация мастер ноды

`curl -sfL https://get.k3s.io | K3S_TOKEN='l%TH]c4VvCT<Xj{' sh -s - server --cluster-init `


Добавляем мастер ноды

`curl -sfL https://get.k3s.io | K3S_TOKEN='l%TH]c4VvCT<Xj{' sh -s - server --server https://192.168.0.5:6443`


Получаем токен на главном мастере

`sudo cat /var/lib/rancher/k3s/server/node-token`

> Не забываем изменить hostname

Добавляем worker

`curl -sfL https://get.k3s.io | K3S_URL="https://192.168.0.5:6443" \
      K3S_TOKEN='K1079038baf34fd74abdb5f5cbc38018cf30b86500f7b28e0934103f23d9cfb8d89::server:l%TH]c4VvCT<Xj{' \
      sh -`


Добавляем роль для worker (наведем чуть красоты)

`sudo kubectl label node srvk3s-worker-1 node-role.kubernetes.io/worker=worker`


INSTALL HELM

`https://helm.sh/docs/intro/install/`

`sudo apt-get install curl gpg apt-transport-https --yes`

`curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null`

`echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list`

`sudo apt-get update`

`sudo apt-get install helm`

Предварительно создаем каталог с проектом и config.yaml 

`https://z2jh.jupyter.org/en/stable/jupyterhub/installation.html`

ПЕРЕБРОСИМ КОНФИГ

`mkdir -p ~/.kube`

`sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config`

`sudo chown (id−u):(id -u):(id−u):(id -g) ~/.kube/config`

`helm upgrade --cleanup-on-fail --install jupyterhub jupyterhub/jupyterhub --namespace jupyterhub --create-namespace --version=3.3.0 --values config.yaml`

`https://tljh.jupyter.org/en/latest/howto/admin/admin-users.html`

## Уставновка jupyterhub на k3s
Склонируем репозиторий helm и обновим его

`helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/`

`helm repo update`

Далее создайте файл конйигурации values.yaml и заполните го следуещем содержимым 

```bash
proxy:
  secretToken: "ЗАМЕНИ_МЕНЯ_НА_32_СИМВОЛА_HEX"  # openssl rand -hex 32
singleuser:
  profileList:
    - display_name: "Default Server with Corporate CA"
      description: "Uses corporate CA via environment variables (no file modification)"
      default: true
      kubespawner_override:
        # --- Внедряем сертификат через initContainer ---
        init_containers:
          - name: inject-corp-ca
            image: ubuntu:22.04
            command: ["/bin/bash", "-c"]
            args:
              - |
                apt-get update && apt-get install -y ca-certificates openssl
                cp /certs/vi.crt /certs-target/
                echo "✅ Corporate CA copied to target volume."
            volumeMounts:
              - name: corp-ca-volume
                mountPath: /certs
                readOnly: true
              - name: certs-target
                mountPath: /certs-target
        volume_mounts:
          - name: certs-target
            mountPath: /etc/custom-certs
        volumes:
          - name: corp-ca-volume
            configMap:
              name: corp-ca-cert
          - name: certs-target
            emptyDir: {}
        environment:
          # Указываем pip, где взять сертификат
          PIP_CERT: "/etc/custom-certs/vi.crt"
          # Указываем Python requests/urllib
          REQUESTS_CA_BUNDLE: "/etc/custom-certs/vi.crt"
          SSL_CERT_FILE: "/etc/custom-certs/vi.crt"
  image:
    name: jupyter/base-notebook
    tag: latest
hub:
  config:
    JupyterHub:
      authenticator_class: firstuse
    FirstUseAuthenticator:
      create_users: true
```


















