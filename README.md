# Matrix Server - LineHost

Este repositório contém a configuração completa para deploy de um servidor Matrix Synapse com cliente Element Web usando Docker Compose no Coolify.

## 🚀 Recursos Incluídos

- **Matrix Synapse** - Servidor Matrix principal
- **PostgreSQL 15** - Banco de dados otimizado
- **Redis 7** - Cache para melhor performance
- **Element Web** - Cliente web moderno
- **Configuração SSL automática** via Coolify
- **Logs estruturados** com rotação automática
- **Backup automático** de dados

## 📋 Pré-requisitos

- Servidor com Coolify instalado
- Domínio configurado (ex: `matrix.linehost.pro`)
- Acesso root ao servidor
- Pelo menos 2GB RAM e 10GB de espaço em disco

## 🛠️ Instalação

### 1. Clone este repositório

```bash
git clone <URL_DO_SEU_REPOSITORIO>
cd matrix-server
```

### 2. Configure as variáveis de ambiente

Copie o arquivo de exemplo e configure suas variáveis:

```bash
cp .env.example .env
nano .env
```

**Importante**: Altere a `POSTGRES_PASSWORD` para uma senha forte!

### 3. Gere chaves secretas

Execute os comandos para gerar as chaves necessárias:

```bash
# Gerar macaroon_secret_key
python3 -c "import secrets; print('Macaroon Secret:', secrets.token_urlsafe(32))"

# Gerar form_secret  
python3 -c "import secrets; print('Form Secret:', secrets.token_urlsafe(32))"
```

Substitua os valores `YOUR_MACAROON_SECRET_KEY_HERE` e `YOUR_FORM_SECRET_HERE` no arquivo `homeserver.yaml`.

### 4. Configure DNS

Adicione os seguintes registros DNS no seu provedor:

```
Tipo  | Nome                        | Valor
------|-----------------------------|-----------------
A     | matrix.linehost.pro         | [IP_DO_SERVIDOR]
A     | element.linehost.pro        | [IP_DO_SERVIDOR]
SRV   | _matrix._tcp.linehost.pro   | 10 0 443 matrix.linehost.pro
```

### 5. Deploy no Coolify

1. Acesse seu painel Coolify
2. Clique em **"New Resource"** → **"Application"**
3. Selecione **"Git Repository"**
4. Cole a URL do repositório
5. Configure os domínios:
   - `matrix.linehost.pro` → porta `8008`
   - `element.linehost.pro` → porta `8080` (opcional)
6. Adicione as variáveis de ambiente na interface do Coolify
7. Clique em **"Deploy"**

## ⚙️ Configuração Pós-Deploy

### Criar usuário administrador

Após o deploy bem-sucedido, crie o primeiro usuário:

```bash
docker exec -it matrix-synapse register_new_matrix_user http://localhost:8008 -c /data/homeserver.yaml --admin
```

### Testar a instalação

1. **Teste da API**: `https://matrix.linehost.pro/_matrix/client/versions`
2. **Cliente Element**: `https://element.linehost.pro`
3. **Teste de Federação**: [Matrix Federation Tester](https://federationtester.matrix.org/)

## 📁 Estrutura de Arquivos

```
matrix-server/
├── docker-compose.yaml              # Configuração principal do Docker
├── .env.example                     # Exemplo de variáveis de ambiente
├── homeserver.yaml                  # Configuração do Matrix Synapse
├── matrix.linehost.pro.log.config  # Configuração de logs
├── element-config.json              # Configuração do cliente Element
└── README.md                        # Esta documentação
```

## 🔧 Personalização

### Alterar domínio

1. Substitua `matrix.linehost.pro` nos arquivos:
   - `docker-compose.yaml`
   - `homeserver.yaml` 
   - `element-config.json`
2. Configure os novos registros DNS
3. Redeploy no Coolify

### Habilitar registro público

Para permitir que usuários se registrem:

```yaml
# No homeserver.yaml
enable_registration: true
enable_registration_without_verification: false
```

### Configurar email (SMTP)

Adicione no `homeserver.yaml`:

```yaml
email:
  smtp_host: "seu-smtp.com"
  smtp_port: 587
  smtp_user: "seu-email@dominio.com"
  smtp_pass: "sua-senha"
  require_transport_security: true
  notif_from: "Matrix <noreply@linehost.pro>"
```

## 📊 Monitoramento

### Logs

```bash
# Logs do Synapse
docker logs matrix-synapse -f

# Logs do PostgreSQL
docker logs matrix-postgres -f

# Logs do Redis
docker logs matrix-redis -f
```

### Métricas

O Synapse expõe métricas Prometheus em `http://localhost:9090/metrics` (apenas internamente).

### Backup

Os dados são persistidos nos volumes Docker:
- `synapse_data` - Dados do Matrix
- `postgres_data` - Banco de dados
- `redis_data` - Cache Redis

## 🛡️ Segurança

### Configurações implementadas:

- ✅ Registro desabilitado por padrão
- ✅ Rate limiting configurado
- ✅ Federação controlada
- ✅ SSL/TLS obrigatório
- ✅ Headers de segurança
- ✅ Logs de auditoria

### Recomendações adicionais:

- Use senhas fortes para todos os serviços
- Mantenha o sistema atualizado
- Configure firewall adequadamente
- Monitore logs regularmente
- Faça backups periódicos

## 🔄 Atualizações

Para atualizar o Matrix Synapse:

```bash
# No diretório do projeto
docker-compose pull
docker-compose up -d
```

O Coolify pode fazer isso automaticamente se configurado.

## 🐛 Solução de Problemas

### Erro de conexão com banco

```bash
# Verificar se PostgreSQL está rodando
docker logs matrix-postgres

# Verificar conectividade
docker exec -it matrix-postgres psql -U synapse -d synapse -c "SELECT version();"
```

### Erro de federação

1. Verifique os registros DNS SRV
2. Teste em: https://federationtester.matrix.org/
3. Verifique se a porta 443 está acessível

### Performance lenta

1. Verifique recursos do servidor
2. Analise logs para erros
3. Considere ajustar cache Redis
4. Otimize configurações do PostgreSQL

## 📞 Suporte

- **Documentação oficial**: https://matrix-org.github.io/synapse/
- **Community**: #synapse:matrix.org
- **Issues**: Use as issues deste repositório

## 📜 Licença

Este projeto está sob a licença Apache 2.0. Veja o arquivo LICENSE para detalhes.

---

**Desenvolvido por LineHost** 🚀

Para mais serviços de hospedagem, visite: https://linehost.pro