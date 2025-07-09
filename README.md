# Aplicação Web com Kubernetes, Rancher Desktop e ArgoCD 🚀

Bem-vindo! 🎉 Este repositório guia você na implantação de uma aplicação de microserviços em um ambiente **Kubernetes** local utilizando o **Rancher Desktop** 🐄. Com práticas de **GitOps** 🛠️, usamos o **ArgoCD** ⚙️ para gerenciar os deploys de forma contínua e automatizada. Todos os manifestos e configurações estão hospedados em um repositório público no **GitHub** 📂, garantindo rastreabilidade, colaboração e automação! 😎

> **Créditos**: Este projeto utiliza a aplicação de microserviços fornecida pelo repositório [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo), desenvolvido pela equipe do Google Cloud Platform.

## 📋 Pré-requisitos

Antes de começar, certifique-se de ter instalado:

- **Git** 🗃️: Para clonar repositórios e gerenciar o código.
- **Rancher Desktop** 🐄: Fornece um ambiente Kubernetes local e suporte ao Docker para criar containers.

Baixe e instale:
- [Git](https://git-scm.com/downloads)
- [Rancher Desktop](https://rancherdesktop.io/)

## 🛠️ Passo a Passo para Configuração

Siga os passos abaixo para implantar a aplicação com sucesso:

### 1. Clonar o Repositório da Aplicação 📥 (Opcional)

Primeiro, clonamos o repositório oficial que contém os manifestos necessários para o deploy:

```bash
git clone https://github.com/GoogleCloudPlatform/microservices-demo
```

- Após o clone, entre no diretório `microservices-demo` e remova todas as pastas, mantendo apenas o arquivo `/release/kubernetes-manifests.yaml`. Este arquivo contém os manifestos necessários para a implantação.

### 2. Instalar e Configurar o ArgoCD no Kubernetes Local ⚙️

O **ArgoCD** será usado para gerenciar os deploys via GitOps. Siga os passos para instalá-lo:

- Crie o namespace `argocd`:

```bash
kubectl create namespace argocd
```

- Instale o ArgoCD aplicando o manifesto oficial:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argocd/stable/manifests/install.yaml
```

- Altere o tipo de serviço do `argocd-server` para `LoadBalancer`:

```bash
kubectl edit svc argocd-server -n argocd
```

No editor que abrir, modifique a propriedade:

De:
```yaml
spec:
  type: ClusterIP
```

Para:
```yaml
spec:
  type: LoadBalancer
```

- Faça o port-forward para acessar a interface gráfica do ArgoCD:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

⚠️ **Não feche o terminal** onde o port-forward está sendo executado. Acesse a interface em [https://localhost:8080](https://localhost:8080).

![Tela de login ArgoCD](/images/img1.png)

- **Credenciais de Login**:
  - **Usuário**: `admin`
  - **Senha**: Para recuperar a senha no Windows, execute:

```powershell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```

Copie o output do comando e use-o como senha na interface do ArgoCD.

### 3. Criar e Sincronizar a Aplicação no ArgoCD 🌟

Agora, configure o **ArgoCD** para gerenciar a aplicação e implantá-la automaticamente:

1. **Acesse a Interface do ArgoCD**:
   - Abra [https://localhost:8080](https://localhost:8080) no navegador e faça login com as credenciais obtidas.
   - Clique em **+ New App** para criar uma nova aplicação.

   ![Botão para criar aplicação](/images/img2.png)

2. **Configurar a Aplicação**:
   - Preencha os campos com as seguintes informações:
     - **Application Name**: `webapp`
     - **Project Name**: `default`
     - **Sync Policy**: Selecione **Manual** (ou **Automatic** para sincronização automática).
     - **Auto-Create Namespace**: Marque esta opção para criar automaticamente o namespace.
     - **Repository URL**: `https://github.com/Thullyoo/gitops-argocd-kubernetes` (Ou o seu repositório com o manifesto)
     - **Revision**: `HEAD`
     - **Path**: `k8s` (ou `/release` se estiver usando o arquivo `kubernetes-manifests.yaml` diretamente).
     - **Cluster URL**: `https://kubernetes.default.svc`
     - **Namespace**: `mywebapp`

   - Clique em **CREATE** para salvar a configuração.

   ![Configurações da aplicação](/images/img3.png)  
   ![Configurações da aplicação - Parte 2](/images/img4.png)

3. **Sincronizar a Aplicação**:
   - Após criar a aplicação, ela aparecerá na interface, mas ainda não estará sincronizada com o repositório.

   ![Aplicação criada, mas não sincronizada](/images/img5.png)

   - Clique na aplicação `webapp` e, em seguida, no botão **SYNCHRONIZE**.

   ![Botão para sincronizar](/images/img6.png)

   - Aguarde alguns minutos enquanto o ArgoCD sincroniza os recursos e implanta a aplicação. A interface mostrará o progresso.

   ![Aplicação sendo sincronizada](/images/img7.png)

4. **Acessar a Aplicação**:
   - Após a sincronização, faça o port-forward do serviço `frontend` para visualizar a aplicação:

   ```bash
   kubectl port-forward svc/frontend -n mywebapp 8081:80
   ```
  ⚠️ **Não feche o terminal** onde o port-forward está sendo executado.
   - Acesse [http://localhost:8081](http://localhost:8081) no navegador para ver a aplicação em ação!

   ![Tela do site](/images/img8.png)



### Bônus: Ajustando Réplicas do Frontend e Testando a Automação do GitOps 🌟

Este bônus demonstra como ajustar a quantidade de réplicas do serviço `frontend` e observar a automação do **GitOps** com **ArgoCD** em ação! 🚀

1. **Editar o Manifesto do Frontend**:
   - Abra o arquivo `kubernetes-manifests.yaml`.
   - Localize o manifesto do **Deployment** do serviço `frontend`.

   ![Manifesto Deployment Frontend](/images/img9.png)

   - Adicione ou altere a propriedade `replicas` na seção `spec` para `2` (ou outro número desejado):

   ```yaml
   spec:
     replicas: 2
   ```

   ![Manifesto com réplicas ajustadas](/images/img10.png)

2. **Fazer o Push da Alteração**:
   - Commit e envie as alterações para o repositório no GitHub:

   ```bash
   git add kubernetes-manifests.yaml
   git commit -m "Aumentar réplicas do frontend para 2"
   git push origin main
   ```

3. **Verificar a Sincronização no ArgoCD**:
   - Após o push, o **ArgoCD** detectará a diferença entre o estado do repositório e o cluster, marcando a aplicação como **OutOfSync** na interface.

   ![Status OutOfSync](/images/img11.png)

   - Acesse a interface do ArgoCD em [https://localhost:8080](https://localhost:8080), clique na aplicação `webapp` e selecione **SYNCHRONIZE** para aplicar as alterações.

4. **Aguardar o Deploy**:
   - O ArgoCD atualizará automaticamente o cluster para refletir o novo número de réplicas. Esse processo pode levar alguns minutos.

5. **Verificar o Resultado**:
   - Confirme que os dois pods do `frontend` foram criados com o comando:

   ```bash
   kubectl get pods -n mywebapp
   ```

   - O output mostrará os pods do `frontend`, confirmando que a automação do GitOps funcionou corretamente.

   ![Pods do frontend criados](/images/img12.png)

## 🎉 Conclusão

Parabéns! 🥳 Você implantou uma aplicação de microserviços em um ambiente **Kubernetes** local, gerenciada pelo **ArgoCD** usando práticas de **GitOps**! Explore a interface do ArgoCD para monitorar os microserviços, verificar o status dos deploys e realizar ajustes. Para mais detalhes, consulte a [documentação oficial do ArgoCD](https://argo-cd.readthedocs.io/) ou o repositório original da aplicação [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo). 🚀

