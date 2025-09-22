# Legacy CI/CD Project: Flask App on Kubernetes
Цей проект демонструє налаштування повного CI/CD пайплайну для деплою Flask додатку в Kubernetes кластері. Зміна в коді автоматично запускає збірку нового Docker-образу та його розгортання в продакшен середовищі.

## Архітектура
1. **Developer**:                 пушить код у GitHub.
2. **GitHub Actions/Build**:      збирає Docker-образ і пушить його в Docker Hub.
3. **GitHub Actions/Deploy**:     підключається до Kubernetes кластера та оновлює Deployment.
4. **Користувач**:                звертається до додатку через доменне ім'я.
5. **DNS (GoDaddy)**:             направляє запит на Kubernetes кластер.
6. **MetalLB**:                   надає кластеру зовнішню IP-адресу.
7. **Nginx Ingress Controller**:  направляє трафік до відповідного Service.
8. **Kubernetes**:                обробляє запит.

![Архітектура](images/1.pdf)

## Як це розгорнути з нуля
**Потрібно**:
- Сервер з Ubuntu 20.04+
- Зареєстрований домен (наприклад, на GoDaddy)

### Крок 1: Налаштування Kubernetes кластера (kubeadm)
1. **Ініціалізувати кластер**
   *Встановити kubeadm, kubelet, kubectl*
2. **Встановити мережевий плагін (Calico)**
3. **Встановити Helm**

### Крок 2: Встановлення Ingress Controller та MetalLB
1. **Встановлення Nginx Ingress Controller**
   ```shell
   helm install my-release oci://ghcr.io/nginx/charts/nginx-ingress --version 2.2.2
   ```
2. **Встановлення MetalLb**
   ```shell
   kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
   ```

  Конфігураційний файл для MetalLb
  ```yaml
    apiVersion: metallb.io/v1beta1
  kind: IPAddressPool
  metadata:
    name: my-ip-pool
    namespace: metallb-system
  spec:
    addresses:
    - 192.168.1.240-192.168.1.249
  ---
  apiVersion: metallb.io/v1beta1
  kind: L2Advertisement
  metadata:
    name: l2-advertisement
    namespace: metallb-system
  ```

### Крок 3: Налаштування DNS
1. **Додати A-запис у панелі керування GoDaddy:**
    - Ім'я: `flask`
    - Значення: `IP-адреса, яку видав MetalLB`

  
### Крок 4: Розгортання додатку
1. **Клонувати репозиторій:**
    ```bash
    git clone https://github.com/Maksumkaa/legacy-cicd-project .
    cd legacy-cicd-project
    ```

2. **Створити namespace та запустити додаток:**
    ```bash
    kubectl create namespace flask
    kubectl apply -f kubernetes/ -n flask
    ```
    
### Крок 5: Перевірка
Перевірити, що все працює:
```bash
kubectl get pods,svc,ingress -n flask
```



### CI/CI Pipeline
**При пуші в гілку main автоматично виконується**
- **Build**: Збірка Docker-образа та пуш на Docker Hub
- **Deploy**: Підключення до K8s кластера та оновлення Deployment

Переглянути workflow можна в [.github/workflows/docker-image.yml](.github/workflows/docker-image.yml)


### Технології
- **Kubernetes (kubeadm)**
- **Docker**
- **GitHub Actions**
- **Nginx Ingress Controller**
- **MetalLB**
- **Flask (Python)**
- **GoDaddy DNS**

## Автор
Саковський Максим - Maksumkaa
