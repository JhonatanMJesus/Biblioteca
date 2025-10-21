# 📚 Sistema de Biblioteca Digital

Sistema completo de gerenciamento de biblioteca desenvolvido em PHP com arquitetura containerizada Docker, proporcionando controle eficiente de livros, empréstimos e usuários com deploy automatizado via CI/CD.

## 📋 Sobre o Projeto

Aplicação web moderna para gestão de acervo bibliográfico, desenvolvida com PHP 8.4 e Apache, totalmente containerizada com Docker Compose. O projeto implementa pipeline CI/CD automatizado através de GitHub Actions, realizando build e deploy contínuo de imagens Docker no Docker Hub a cada commit na branch principal.

## ✨ Funcionalidades

- ✅ **Gestão de Livros**: Cadastro e controle de acervo completo
- ✅ **Empréstimos**: Sistema de controle de empréstimos e devoluções
- ✅ **Usuários**: Gerenciamento de cadastros de leitores
- ✅ **Containerização**: Ambiente isolado e replicável com Docker
- ✅ **CI/CD**: Deploy automatizado via GitHub Actions
- ✅ **Persistência**: Volumes Docker para dados permanentes
- ✅ **Escalabilidade**: Arquitetura multi-container orquestrada

## 🛠️ Tecnologias Utilizadas

### Backend & Runtime
- **Linguagem**: PHP 8.4
- **Servidor Web**: Apache HTTP Server
- **Extensões PHP**: MySQLi, PDO, PDO MySQL
- **Mod Rewrite**: Apache mod_rewrite habilitado

### Banco de Dados
- **SGBD**: MySQL 8.0
- **Persistência**: Docker Named Volumes

### DevOps & Infraestrutura
- **Containerização**: Docker & Docker Compose
- **CI/CD**: GitHub Actions
- **Registry**: Docker Hub
- **Build Tool**: Docker Buildx (multi-platform)

### Ferramentas
- **Orquestração**: Docker Compose
- **Automação**: GitHub Workflows
- **Secrets Management**: GitHub Secrets

## 📁 Estrutura do Projeto
```
Biblioteca/
├── .github/
│   └── workflows/
│       └── docker-build.yml      # Pipeline CI/CD
├── App/                          # Código-fonte da aplicação
│   ├── controllers/
│   ├── models/
│   ├── views/
│   └── config/
├── Dockerfile                    # Configuração da imagem PHP
├── docker-compose.yml            # Orquestração de containers
├── index.php                     # Ponto de entrada
├── .htaccess                     # Configurações Apache
└── README.md
```

## 🐳 Arquitetura Docker

### Serviços

#### PHP Container (`meu_servidor_php`)
```yaml
- Imagem: PHP 8.4 com Apache
- Porta: 8080:80
- Extensões: mysqli, pdo, pdo_mysql
- Mod Rewrite: Habilitado
- Dependência: MySQL Container
```

#### MySQL Container (`meu_servidor_mysql`)
```yaml
- Imagem: MySQL 8.0
- Porta: 3307:3306 (evita conflito com MySQL local)
- Database: db_biblioteca
- Usuário: user / senha
- Root Password: root
- Volume: mysql-data (persistência)
```

### Network
Os containers compartilham rede interna Docker, permitindo comunicação via hostname:
- PHP acessa MySQL via: `mysql:3306`

## 🚀 Como Executar

### Pré-requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop) ou Docker Engine
- [Docker Compose](https://docs.docker.com/compose/install/) (geralmente incluído no Docker Desktop)
- Git

### Instalação Local

1. **Clone o repositório**
```bash
git clone https://github.com/seu-usuario/biblioteca.git
cd biblioteca
```

2. **Inicie os containers**
```bash
docker-compose up -d
```

3. **Verifique os containers**
```bash
docker-compose ps
```

4. **Acesse a aplicação**
```
http://localhost:8080
```

5. **Acesse o banco de dados** (opcional)
```bash
# Via linha de comando
docker exec -it meu_servidor_mysql mysql -u user -psenha db_biblioteca

# Via cliente MySQL (ex: MySQL Workbench)
Host: localhost
Porta: 3307
Usuário: user
Senha: senha
Database: db_biblioteca
```

### Parar os Containers
```bash
# Parar sem remover
docker-compose stop

# Parar e remover containers (mantém volumes)
docker-compose down

# Parar, remover containers E volumes
docker-compose down -v
```

## 🔧 Comandos Úteis

### Docker Compose
```bash
# Iniciar em modo attached (ver logs)
docker-compose up

# Reconstruir imagens e iniciar
docker-compose up --build

# Ver logs em tempo real
docker-compose logs -f

# Ver logs de um serviço específico
docker-compose logs -f php
docker-compose logs -f mysql

# Executar comandos dentro do container
docker-compose exec php bash
docker-compose exec mysql bash

# Reiniciar serviço específico
docker-compose restart php
```

### Docker
```bash
# Listar containers em execução
docker ps

# Listar todas as imagens
docker images

# Remover imagens não utilizadas
docker image prune -a

# Ver uso de volumes
docker volume ls

# Inspecionar volume
docker volume inspect biblioteca_mysql-data

# Limpar sistema completo
docker system prune -a --volumes
```

### MySQL
```bash
# Backup do banco de dados
docker exec meu_servidor_mysql mysqldump -u user -psenha db_biblioteca > backup.sql

# Restaurar banco de dados
docker exec -i meu_servidor_mysql mysql -u user -psenha db_biblioteca < backup.sql

# Acessar MySQL CLI
docker exec -it meu_servidor_mysql mysql -u root -proot
```

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

O projeto implementa pipeline automatizado que:

1. **Trigger**: Executa em push na branch `main` ou manualmente via `workflow_dispatch`
2. **Checkout**: Clona o repositório
3. **Login DockerHub**: Autentica usando secrets
4. **Setup Buildx**: Configura builder para builds otimizados
5. **Build & Push**: Constrói imagem e envia para Docker Hub

### Configuração de Secrets

No GitHub, configure em **Settings → Secrets and variables → Actions**:
```
DOCKERHUB_USERNAME: seu-usuario-dockerhub
DOCKERHUB_TOKEN: seu-token-de-acesso
```

### Usar Imagem do Docker Hub
```bash
# Pull da imagem
docker pull seu-usuario/biblioteca_php:latest

# Executar container
docker run -d -p 8080:80 seu-usuario/biblioteca_php:latest
```

## 📦 Dockerfile Explicado
```dockerfile
# Imagem base PHP 8.4 com Apache
FROM php:8.4-apache

# Instala extensões MySQL necessárias
RUN docker-php-ext-install mysqli pdo pdo_mysql

# Habilita mod_rewrite para URLs amigáveis
RUN a2enmod rewrite

# Copia arquivos da aplicação para o container
COPY ./App /var/www/html/App/
COPY ./index.php /var/www/html/
COPY ./.htaccess /var/www/html/

# Expõe porta 80 (padrão Apache)
EXPOSE 80
```

## 🗄️ Configuração do Banco de Dados

### Variáveis de Ambiente MySQL
```yaml
MYSQL_ROOT_PASSWORD: root          # Senha do usuário root
MYSQL_DATABASE: db_biblioteca      # Database criado automaticamente
MYSQL_USER: user                   # Usuário adicional
MYSQL_PASSWORD: senha              # Senha do usuário
```

### String de Conexão PHP
```php
<?php
$host = 'mysql';  // Nome do container (via Docker network)
$db   = 'db_biblioteca';
$user = 'user';
$pass = 'senha';
$port = 3306;     // Porta interna do container

$dsn = "mysql:host=$host;port=$port;dbname=$db;charset=utf8mb4";

try {
    $pdo = new PDO($dsn, $user, $pass, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false
    ]);
} catch (PDOException $e) {
    die("Erro de conexão: " . $e->getMessage());
}
?>
```

## 🔒 Segurança

### Boas Práticas Implementadas

✅ **Senhas não hardcoded**: Usar variáveis de ambiente em produção  
✅ **Portas customizadas**: 3307 externa evita conflitos  
✅ **Network isolada**: Containers em rede privada Docker  
✅ **Volumes nomeados**: Persistência segura de dados  
✅ **Token no Registry**: Autenticação via token, não senha  

### Recomendações para Produção

- [ ] Use `.env` file para credenciais
- [ ] Implemente SSL/TLS (HTTPS)
- [ ] Configure firewall adequado
- [ ] Use secrets do Docker Swarm/Kubernetes
- [ ] Implemente backup automatizado
- [ ] Configure health checks
- [ ] Use imagens slim ou alpine
- [ ] Scaneie vulnerabilidades (Trivy, Snyk)

## 🚀 Deploy em Produção

### Opção 1: Docker Compose em VPS
```bash
# No servidor
git clone https://github.com/seu-usuario/biblioteca.git
cd biblioteca

# Configure .env
cp .env.example .env
nano .env

# Inicie com restart policy
docker-compose up -d --restart unless-stopped
```

### Opção 2: Docker Swarm
```bash
docker stack deploy -c docker-compose.yml biblioteca
```

### Opção 3: Kubernetes
```bash
# Converter docker-compose para k8s
kompose convert

# Deploy
kubectl apply -f .
```

## 📊 Monitoramento

### Logs
```bash
# Ver logs de todos os serviços
docker-compose logs -f

# Logs com timestamp
docker-compose logs -f -t

# Últimas 100 linhas
docker-compose logs --tail=100
```

### Health Check
```bash
# Status dos containers
docker-compose ps

# Estatísticas de uso
docker stats

# Inspecionar container
docker inspect meu_servidor_php
```

## 🧪 Testes

### Testar Conexão PHP-MySQL
```bash
# Executar dentro do container PHP
docker-compose exec php php -r "new PDO('mysql:host=mysql;dbname=db_biblioteca', 'user', 'senha');"
```

### Testar Extensões PHP
```bash
docker-compose exec php php -m | grep -i mysql
# Deve mostrar: mysqli, mysqlnd, pdo_mysql
```

## 🐛 Troubleshooting

### Container não inicia
```bash
# Ver logs detalhados
docker-compose logs php
docker-compose logs mysql

# Verificar portas em uso
netstat -ano | findstr :8080
netstat -ano | findstr :3307
```

### Erro de conexão MySQL
```bash
# Aguardar MySQL inicializar completamente (primeira execução)
docker-compose logs mysql | grep "ready for connections"

# Testar conexão
docker-compose exec mysql mysql -u user -psenha -e "SELECT 1"
```

### Volume corrompido
```bash
# Remover volume e recriar
docker-compose down -v
docker-compose up -d
```

## 📝 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## 👥 Contribuindo

Contribuições são bem-vindas! Para contribuir:

1. Fork o projeto
2. Crie uma branch (`git checkout -b feature/NovaFuncionalidade`)
3. Commit suas mudanças (`git commit -m 'Adiciona nova funcionalidade'`)
4. Push para a branch (`git push origin feature/NovaFuncionalidade`)
5. Abra um Pull Request

## 📞 Contato

**Desenvolvedor**: Seu Nome
- Email: seu.email@exemplo.com
- LinkedIn: [seu-perfil](https://linkedin.com/in/seu-perfil)
- GitHub: [@seu-usuario](https://github.com/seu-usuario)
- Docker Hub: [seu-usuario](https://hub.docker.com/u/seu-usuario)

---

⭐ **Desenvolvido com Docker** | 🐳 **Containerizado e pronto para produção**
