# ☁️ Lab DevOps: Do Código à Nuvem com Docker e AWS

Olá! Este repositório documenta a minha jornada prática construindo uma infraestrutura em nuvem do zero. Como estagiário de projetos atuando ativamente com GMUD (Gestão de Mudanças), decidi colocar a "mão na massa" para entender profundamente como as entregas de software funcionam por baixo dos panos, unindo a disciplina de processos com a engenharia DevOps.

Neste laboratório, peguei um site estático simples e criei todo o pipeline de containerização e hospedagem utilizando **Docker** e a infraestrutura da **AWS**.

## 🎯 O Objetivo do Projeto
Criar um ambiente confiável, versionado e escalável para hospedar uma página web, garantindo que o que funciona na minha máquina local funcione perfeitamente no servidor em produção.

## 🧰 Minha Caixa de Ferramentas
| Ferramenta | O que fez neste projeto? |
| :--- | :--- |
| **Docker** | Isolou o site e o servidor web em um container (garantia de "funciona em qualquer lugar"). |
| **Nginx** | O motor do servidor web, responsável por entregar as páginas de forma ultra-rápida. |
| **Amazon ECR** | O "cofre" seguro na nuvem onde versionei e guardei minha imagem Docker. |
| **Amazon EC2** | A máquina virtual (Linux) que serviu como host para rodar o projeto para o mundo. |
| **Linux (Fedora/WSL)** | Meu ambiente de desenvolvimento e controle de terminal. |

---

## 🧠 Diário de Bordo: Desafios e Soluções

A teoria é linda, mas a prática ensina muito mais. Durante o deploy, enfrentei e resolvi alguns gargalos técnicos interessantes:

### 1. O Bug Silencioso do Nginx (`daemon off`)
Logo no primeiro build, meu container subia e "morria" instantaneamente. Ao investigar os logs, encontrei o erro: `nginx: [emerg] unexpected end of parameter, expecting ";"`. 
> **A Solução:** Fui direto no `Dockerfile` e identifiquei um erro de sintaxe clássico. O comando `CMD` precisava do ponto e vírgula no final. Corrigi para `CMD ["nginx", "-g", "daemon off;"]` e apliquei os conceitos de GMUD para gerar uma nova versão da imagem (`v1.1`), mantendo o histórico de alterações limpo.

### 2. A Batalha das Portas e o Firewall (Security Groups)
Inicialmente, mapeei o site para a porta `81`. O container rodava perfeitamente na AWS, mas não abria no navegador. 
> **A Solução:** Entendi que a AWS opera sob o conceito de "negação por padrão". Acessei as configurações do **Security Group** da EC2, liberei o tráfego de entrada (Inbound Rules) e, para seguir as melhores práticas de mercado, reconfigurei o container para rodar na porta HTTP padrão (`80`). Assim, o site ficou acessível diretamente pelo IP, sem necessidade de especificar portas na URL.

## 🔐 Segurança e Governança (IAM)

Um ponto crucial para o sucesso do deploy foi a configuração de permissões na AWS. Por padrão, uma instância EC2 não tem autorização para "conversar" com o ECR. 

Para resolver isso de forma segura (seguindo o princípio do privilégio mínimo), realizei as seguintes configurações:

1. **Criação de uma IAM Role**: Configurei uma Role específica para instâncias EC2.
2. **Permissão de Leitura (Policy)**: Anexei a política `AmazonEC2ContainerRegistryReadOnly`.
3. **Associação de Perfil**: Vinculei essa Role à minha instância EC2 via console AWS (**Actions > Security > Modify IAM Role**).

> **Por que isso é importante?** Sem essa configuração, o comando `docker pull` na EC2 retornaria um erro de `Access Denied`. Ao usar Roles em vez de chaves de acesso fixas (`access keys`), garantimos que as credenciais nunca fiquem expostas dentro do servidor.
---

## 🚀 Passo a Passo da Execução (Como reproduzir)

Se você quiser recriar este laboratório, aqui está o caminho das pedras que eu utilizei:

## Build da imagem (Versão 1.1)
docker build -t meu-site:v1.1 .

## Rodando localmente para validar (Acessível em localhost:8080)
docker run -d -p 8080:80 --name teste-local meu-site:v1.1

### Deploy na Nuvem: ECR e EC2

**2. Login seguro e envio para a AWS (ECR):**
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin [ID_DA_SUA_CONTA_AWS].dkr.ecr.us-east-1.amazonaws.com

docker tag meu-site:v1.1 [ID_DA_SUA_IMAGEM_ECR]:v1.1

docker push [ID_DA_SUA_IMAGEM_ECR]:v1.1
```
**3. O Deploy na Máquina Virtual (EC2):**
Após conectar na EC2 via protocolo SSH (.pem), executei o container mapeando a porta web oficial:

docker run -d -p 80:80 --name site-prod [ID_DA_SUA_IMAGEM_ECR]:v1.1

## 📸 Evidências do Laboratório

Aqui estão os registros do ambiente rodando com sucesso antes do decommissioning (desligamento por controle de custos).
O Site no Ar (Acesso Externo)
<img width="1651" height="201" alt="Screenshot_1" src="https://github.com/user-attachments/assets/9bf8d643-6f5d-4e22-ab90-38dcc11113d3" />
<img width="1902" height="1001" alt="Screenshot_3" src="https://github.com/user-attachments/assets/b797a872-0c74-4aba-90c0-65e3c33fb984" />

### ⭐ Projeto da Maria Lazara:
https://www.linkedin.com/in/marialazaradev/
https://www.youtube.com/@marialazaradev
