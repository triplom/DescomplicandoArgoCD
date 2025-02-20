# 🚀 Descomplicando ArgoCD - DAY-1

Seja bem-vindo ao **Day-1** do Descomplicando ArgoCD! Hoje você vai embarcar em uma jornada para entender o ArgoCD, GitOps e os conceitos de Continuous Delivery no Kubernetes. Preparado para essa aventura? 😃

---

## 📚 O que vamos aprender no Day-1?

- **Conceitos Básicos:** O que é ArgoCD, GitOps e o conceito de *estado desejado*.
- **Configuração do Ambiente:** Como instalar o ArgoCD como operador no Kubernetes e o CLI para gerenciá-lo.
- **Autenticação e Acesso:** Passos para se autenticar no ArgoCD e adicionar seu cluster.
- **Criação e Sincronização de Aplicações:** Como criar sua primeira aplicação (app) no ArgoCD e sincronizá-la com seu repositório Git.
- **Monitoramento e Logs:** Aprenda a verificar o status e os logs da sua aplicação.

---

## 📖 Conteúdo do Guia

1. [Introdução ao ArgoCD](#-o-que-é-o-argocd)
2. [Entendendo o GitOps](#-o-que-é-gitops)
3. [Pré-requisitos](#-pré-requisitos)
4. [Instalação do ArgoCD](#️-instalando-o-argocd)
   - [Operador no Kubernetes](#instalando-o-argocd-como-operador-no-kubernetes)
   - [ArgoCD CLI](#instalando-o-argocd-cli)
5. [Autenticando no ArgoCD](#-autenticando-no-argocd)
6. [Adicionando seu Cluster](#-adicionando-seu-cluster-ao-argocd)
7. [Criando a Aplicação no ArgoCD](#-criando-a-aplicação-no-argocd)
   - [Criação da App Exemplo](#criando-a-app-exemplo)
   - [Sincronizando e Monitorando a App](#sincronizando-e-monitorando-a-aplicação)
8. [Recapitulação e Final do Day-1](#-recapitulação-do-day-1)

---

## 🤔 O que é o ArgoCD?

O **ArgoCD** é uma ferramenta open source que facilita o gerenciamento de aplicações no Kubernetes utilizando o GitOps. Ele é escrito em Go e utiliza o [Kubernetes Operator Pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) para orquestrar os recursos do cluster.

👉 **Principais Benefícios:**

- Gerencia o *estado desejado* das aplicações.
- Separa a entrega contínua (CD) do processo de integração contínua (CI).
- Amplamente utilizado nas melhores práticas de DevOps em grandes empresas.

---

## 🔄 O que é GitOps?

GitOps é uma metodologia na qual o Git é a única fonte de verdade para a configuração do seu ambiente. Em vez de aplicar mudanças diretamente, você declara o estado desejado da sua aplicação em arquivos YAML e deixa o ArgoCD e o Kubernetes fazerem o resto. Assim, o que está no Git é o que é executado no cluster! 🚀

---

## ✅ Pré-requisitos

Antes de iniciar, verifique se você possui:

- Um cluster Kubernetes rodando 🌀
- `kubectl` instalado e configurado 🔧
- Muita vontade de aprender e experimentar! 💪

---

## 🛠️ Instalando o ArgoCD

### Instalando o ArgoCD como Operador no Kubernetes

1. **Crie a Namespace:**

   ```bash
   kubectl create namespace argocd
   ```

2. **Instale o ArgoCD:**

   ```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Verifique os Pods:**

   ```bash
   kubectl get pods -n argocd
   ```

   Você deverá ver todos os pods do ArgoCD com o status `Running`. 🎉

### Instalando o ArgoCD CLI

1. **Baixe o binário:**

   ```bash
   curl -SL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
   ```

2. **Instale o binário:**

   ```bash
   sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
   rm argocd-linux-amd64
   ```

3. **Verifique a versão:**

   ```bash
   argocd version
   ```

---

## 🔐 Autenticando no ArgoCD

1. **Identifique o endereço do serviço:**

   ```bash
   kubectl get svc -n argocd
   ```

   Procure pelo serviço `argocd-server`.

2. **Faça o Port-Forward:**

   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

3. **Realize o Login:**

   ```bash
   argocd login localhost:8080
   ```

   - Usuário padrão: `admin`
   - Para obter a senha inicial, execute:

     ```bash
     argocd admin initial-password -n argocd
     ```

4. **Recomenda-se alterar a senha inicial:**

   ```bash
   argocd account update-password
   ```

---

## 🌐 Adicionando seu Cluster ao ArgoCD

1. **Verifique seu contexto atual:**

   ```bash
   kubectl config current-context
   ```

2. **Adicione o cluster:**

   ```bash
   argocd cluster add <NOME_DO_SEU_CONTEXT>
   ```

   Exemplo:

   ```bash
   argocd cluster add girus@eks-cluster.us-east-1.eksctl.io
   ```

3. **Confirme a adição:**

   ```bash
   argocd cluster list
   ```

---

## 📝 Criando a Aplicação no ArgoCD

### Criando a App Exemplo

Utilize um repositório Git com os manifestos necessários. No nosso exemplo, usamos o repositório [k8s-deploy-nginx-example](https://github.com/badtuxx/k8s-deploy-nginx-example) que contém:

- `nginx-deployment.yaml`
- `nginx-service.yaml`
- `nginx-configmap.yaml`
- `nginx-pod.yaml`

### Criando a App via CLI

Execute o comando abaixo para criar a aplicação:

```bash
argocd app create nginx-app \
  --repo https://github.com/badtuxx/k8s-deploy-nginx-example.git \
  --path . \
  --dest-server https://<SEU_CLUSTER_API> \
  --dest-namespace default
```

> **Dica:** 💡  Substitua `<SEU_CLUSTER_API>` pelo endereço do seu cluster.

### Sincronizando e Monitorando a Aplicação

1. **Liste as aplicações:**

   ```bash
   argocd app list
   ```

2. **Verifique o status:**

   ```bash
   argocd app get nginx-app
   ```

   Caso o status esteja como `OutOfSync`, sincronize a aplicação:

   ```bash
   argocd app sync nginx-app
   ```

3. **Confira novamente para confirmar que todos os recursos estão `Synced` e `Healthy`:**

   ```bash
   argocd app get nginx-app
   ```

4. **Visualize os logs da aplicação (por exemplo, do container nginx):**

   ```bash
   argocd app logs nginx-app --container nginx
   ```

---

## 🔄 Recapitulação do Day-1

Hoje você aprendeu:

- Como instalar e configurar o ArgoCD no Kubernetes.
- O conceito de GitOps e o que significa declarar o *estado desejado*.
- Como autenticar e acessar a interface do ArgoCD via CLI.
- Adicionar seu cluster ao ArgoCD.
- Criar, sincronizar e monitorar uma aplicação com ArgoCD.

Cada passo dado é um avanço na sua jornada de Continuous Delivery e GitOps. Continue praticando e explorando! 🚀

---

## 💬 Feedback e Contribuição

Sua opinião é muito importante! Se tiver dúvidas, sugestões ou quiser contribuir, abra uma issue ou envie um pull request. Juntos, podemos tornar este guia ainda melhor! 🤝

---

Esperamos que este guia facilite sua experiência com ArgoCD e que você se divirta nessa jornada de aprendizado. Vamos em frente e #VAIIII! 😃

---
