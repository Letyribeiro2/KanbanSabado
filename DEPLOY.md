# 🚀 Guia de Deploy no Render - Quadro Kanban

Este documento serve como um dossiê completo contendo o passo a passo para a realização de um deploy com sucesso deste projeto na plataforma **Render**.

---

## 🛠️ Modificações Realizadas no Projeto

Para garantir a compatibilidade com o ambiente de produção do Render, padronizamos os seguintes parâmetros no código-fonte:

1. **Porta Dinâmica (`server.js`)**:
   - Substituímos a porta estática `3000` por `process.env.PORT || 3000`. O Render define dinamicamente a porta em que o servidor deve escutar através da variável `PORT`.
2. **Chave Secreta do JWT (`server.js`)**:
   - Permitimos a definição da chave através de `process.env.JWT_SECRET`, mantendo uma chave de desenvolvimento como fallback.
3. **Persistência de Dados (`server.js`)**:
   - Criamos suporte à variável de ambiente `process.env.DATA_DIR`. Como o sistema de arquivos do Render é efêmero, os arquivos JSON locais seriam deletados em cada reinicialização ou deploy. Com isso, os arquivos de dados (`users.json`, `tasks.json`, `revoked_tokens.json`) são armazenados no diretório configurado, permitindo o uso de um **Persistent Disk** (Disco Persistente) do Render.
4. **URL de API Dinâmica (`public/index.html`)**:
   - Alteramos a URL da API do frontend (`API_URL`) de `http://localhost:3000/api` para `window.location.origin + '/api'`. Isso garante que o frontend faça requisições automáticas para o mesmo domínio em que está hospedado, eliminando problemas de CORS ou URLs absolutas incorretas tanto em desenvolvimento local quanto em produção.

---

## 📋 Pré-requisitos
- Uma conta na plataforma [Render](https://render.com/).
- O código deste projeto enviado para um repositório no seu **GitHub** ou **GitLab**.

---

## 🚀 Passo a Passo para o Deploy no Render

### Passo 1: Criar um Novo Web Service
1. Faça login no seu painel do **Render**.
2. Clique no botão **New +** no canto superior direito e selecione a opção **Web Service**.
3. Conecte sua conta do GitHub/GitLab (se ainda não o fez) e selecione o repositório deste projeto.

### Passo 2: Configurar Parâmetros Básicos do Web Service
Preencha os seguintes campos na página de criação:
- **Name**: `kanban-sys` (ou o nome que você preferir)
- **Region**: Selecione a região geograficamente mais próxima de você (ex: `Oregon (us-west-2)` ou `Frankfurt (eu-central-1)`)
- **Branch**: `main` (ou a branch principal que contém o código do projeto)
- **Root Directory**: `kanbansabado` (isso é extremamente importante, pois a pasta do projeto está dentro do subdiretório `kanbansabado` no repositório)
- **Runtime**: `Node`
- **Build Command**: `npm install`
- **Start Command**: `npm start`
- **Instance Type**: `Free` (ou o plano de sua preferência)

### Passo 3: Configurar Variáveis de Ambiente (Environment)
1. Vá até a seção **Environment Variables** (ou clique na aba **Environment** após a criação).
2. Adicione as seguintes chaves e valores:

| Variável | Valor Recomendado | Descrição |
| :--- | :--- | :--- |
| `NODE_ENV` | `production` | Define o ambiente como produção |
| `JWT_SECRET` | _[Uma chave forte e aleatória]_ | Chave usada para assinar os tokens JWT (ex: `c8d19762fc3bb1e17...`) |
| `DATA_DIR` | `/data` | Caminho do volume onde os dados do Kanban serão persistidos (para persistência permanente) |

*(Para gerar uma chave forte para o `JWT_SECRET`, você pode usar o comando `openssl rand -base64 32` no seu terminal local ou digitar uma sequência longa de caracteres aleatórios).*

### Passo 4: Configurar Persistência de Dados (Evitando a perda de tarefas e usuários)
Como o Render utiliza containers efêmeros, qualquer alteração nos arquivos JSON (`users.json`, `tasks.json`) é desfeita quando o serviço é reiniciado (o que ocorre pelo menos uma vez por dia ou a cada novo deploy).

Para contornar isso e persistir os seus usuários e tarefas usando o nosso suporte a `DATA_DIR`:
1. Após criar o Web Service, acesse a página dele no painel do Render.
2. No menu lateral esquerdo, clique em **Disks**.
3. Clique em **Add Disk**.
4. Configure o disco da seguinte forma:
   - **Name**: `kanban-data`
   - **Mount Path**: `/data`
   - **Size**: `1 GiB` (o suficiente para milhares de usuários e tarefas em formato JSON)
5. Certifique-se de que a variável de ambiente `DATA_DIR` está configurada como `/data` na aba **Environment**.
6. Salve as alterações. O Render irá reiniciar o serviço montando o disco persistente. Agora os dados estarão salvos permanentemente!

---

## 🔍 Como Testar e Validar
Após o deploy concluir com sucesso, o Render fornecerá uma URL pública (ex: `https://kanban-sys.onrender.com`).
1. Acesse essa URL no seu navegador.
2. Tente registrar um novo usuário.
3. Faça o login e adicione algumas tarefas no quadro.
4. Mova-as entre colunas (A Fazer, Fazendo, Concluído) e verifique se as alterações são salvas ao atualizar a página.
5. Para testar a persistência, realize um novo deploy manual no painel do Render ("Manual Deploy" -> "Clear cache and deploy") e verifique se as suas tarefas e usuários continuam lá após o reinício!

---
*Dossiê gerado em 23/05/2026.*
