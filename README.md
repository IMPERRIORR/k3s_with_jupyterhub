# Инструкция по уставновке k3s и jupterhub.
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



Не забываем изменить hostname

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

Предварительно создаем каталог с проектом
и config.yaml согласно

`https://z2jh.jupyter.org/en/stable/jupyterhub/installation.html`

ПЕРЕБРОСИМ КОНФИГ

`mkdir -p ~/.kube`

`sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config`

`sudo chown (id−u):(id -u):(id−u):(id -g) ~/.kube/config`

`helm upgrade --cleanup-on-fail --install jupyterhub jupyterhub/jupyterhub --namespace jupyterhub --create-namespace --version=3.3.0 --values config.yaml`

`https://tljh.jupyter.org/en/latest/howto/admin/admin-users.html`

