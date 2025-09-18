# Guia de Atualização do Fork Yii 1 - Empresa

## 📋 Pré-requisitos

- Git configurado e acesso aos repositórios
- Acesso ao GitHub com permissões adequadas
- Acesso ao Packagist com conta vinculada ao GitHub
- Composer instalado localmente

## 🔄 Parte 1: Atualizar o Fork (Repositório analiseeditorial/yii-analise)

### 1. Configurar o upstream (apenas na primeira vez)

```bash
# Clonar o fork (se ainda não tiver)
git clone https://github.com/analiseeditorial/yii-analise.git
cd yii-analise

# Adicionar o repositório upstream do Yii oficial
git remote add upstream https://github.com/yiisoft/yii.git

# Verificar remotes configurados
git remote -v
```

### 2. Criar branch para atualização (NUNCA direto no master!)

```bash
# Buscar as últimas alterações do upstream
git fetch upstream

# Criar branch de atualização a partir do master atual
git checkout master
git pull origin master
git checkout -b feature/sync-upstream

# Fazer merge das alterações do upstream NA BRANCH
git merge upstream/master

# Resolver conflitos se houver (editar arquivos manualmente)
# Depois de resolver:
git add .
git commit -m "Resolve merge conflicts with upstream Yii 1.1.xx"

# Push da branch de atualização
git push origin feature/sync-upstream
```

### 3. Testar a atualização (ANTES de publicar no Packagist)

```bash

# No projeto legacy:
# Adicionar no composer.json do projeto:
# "repositories": [
#     {
#         "type": "vcs", 
#         "url": "https://github.com/analiseeditorial/yii-analise"
#     }
# ],
# Alterar require para: "analiseeditorial/yii": "dev-feature/sync-upstream"

# Backup e teste:
cp composer.lock composer.lock.backup
composer update analiseeditorial/yii --prefer-source

# Verificar se está usando a branch correta:
composer show analiseeditorial/yii
# Deve mostrar: source : [git] https://github.com/analiseeditorial/yii-analise.git

# Testar funcionalidades críticas manualmente

# Se der problema, restaurar:
# cp composer.lock.backup composer.lock && composer install
```

### 4. Criar Pull Request no próprio fork

1. Ir para https://github.com/analiseeditorial/yii-analise
2. Clicar em **"Compare & pull request"**
3. **Base**: `master` ← **Compare**: `feature/sync-upstream`
4. **Título**: `Sync with upstream Yii 1.1.xx`
5. **Descrição**:
   ```markdown
   ## Sync com Upstream Yii
   - Sincronização com yiisoft/yii versão 1.1.28
   - Conflitos resolvidos em: [listar arquivos se houver]
   - Testes realizados: ✅
   
   ## Checklist
   - [ ] Sem conflitos de merge
   - [ ] Funcionalidades críticas testadas
   - [ ] Documentação atualizada (se necessário)
   ```

### 5. Review e Merge após validação

```bash
# Após aprovação do PR, fazer merge no GitHub OU:

# Localmente (se preferir):
git checkout master
git pull origin master
git merge feature/sync-upstream
git push origin master

# Limpar branch após merge
git branch -d feature/sync-upstream
git push origin --delete feature/sync-upstream
```

### 6. Criar tag versionada (APENAS após merge no master)

```bash
# Atualizar master local após merge
git checkout master
git pull origin master

# Criar tag localmente (incrementar versão conforme necessário)
git tag -a 1.1.xx-p1 -m "Update to Yii 1.1.xx - Patch 1"

# Push da tag
git push origin 1.1.xx-p1

# Listar tags para confirmar
git tag -l
```

## 📦 Parte 2: Publicar no Packagist (APENAS após testes aprovados)

### 1. Acessar Packagist

1. Ir para https://packagist.org/
2. Fazer login com conta do GitHub
3. Navegar para **Nome do perfil > My Packages**
4. Localizar **analiseeditorial/yii**

### 2. Sincronizar pacote

1. Clicar em **"Update"** para sincronizar
2. Aguardar processamento
3. Verificar **"View Details & Stats"** para confirmar nova versão
4. Conferir se a tag `1.1.xx-p1` aparece na lista de versões

## 🚀 Parte 3: Atualizar Projeto Legacy

### 1. Criar branch de atualização no projeto legacy

```bash
# No repositório do projeto legacy
cd /var/www/legacy

# Criar branch para atualização
git checkout -b feature/update-yii-1.1.xx-p1

```

### 2. Atualizar composer.json

```bash
# Editar composer.json
# Alterar versão do yii:
# "analiseeditorial/yii": "1.1.xx-p1"

```

### 3. Atualizar dependências

```bash
# Limpar cache do composer (opcional)
composer clear-cache

# Atualizar apenas o pacote yii
composer update analiseeditorial/yii --with-dependencies

# Ou atualizar todas as dependências (cuidado!)
# composer update
```

### 4. Validação e testes

```bash
# só testa
```

### 5. Commit e Pull Request

```bash
# Adicionar alterações
git add composer.json composer.lock

# Commit com mensagem descritiva
git commit -m "Update Yii framework to version 1.1.xx-p1"

# Push da branch
git push origin feature/update-yii-1.1.xx-p1
```

### 6. Criar Pull Request no GitHub

1. Ir para repositório do projeto legacy no GitHub
2. Clicar em **"Compare & pull request"**
3. **Título**: `Update Yii framework to 1.1.xx-p1`
4. **Descrição**:
   ```markdown
   ## Descrição
   Atualização do framework Yii de 1.1.x-p1 para 1.1.28-p1
   
   ## Alterações
   - ✅ Fork sincronizado com upstream
   - ✅ Nova tag 1.1.28-p1 criada
   - ✅ Packagist atualizado
   - ✅ Composer.json atualizado
   - ✅ Testes executados com sucesso
   ```

5. Assignar reviewers apropriados
6. Aguardar aprovação e fazer merge

# 🚨*ATENÇÃO: A PARTIR DAQUI FOI UMA IA QUE GEROU, NÃO FOI REVISADO*


## 🚨 Plano de Rollback (caso algo dê errado)

### Se a sincronização causar problemas:

```bash
# 1. Reverter o merge no master (se já foi feito)
git checkout master
git log --oneline -10  # encontrar hash do commit antes do merge
git reset --hard <hash_commit_anterior>
git push origin master --force-with-lease

# 2. Ou criar hotfix a partir da tag anterior
git checkout 1.1.27-p1  # versão anterior estável
git checkout -b hotfix/rollback-to-1.1.27-p1
# fazer correções necessárias
git tag -a 1.1.28-p2 -m "Rollback to stable 1.1.27 base"
git push origin hotfix/rollback-to-1.1.27-p1
git push origin 1.1.28-p2
```

### Se o Packagist não aceitar a versão:
- Aguardar sincronização automática (até 1 hora)
- Verificar se a tag existe no GitHub
- Contatar suporte do Packagist se necessário

## 🔍 Troubleshooting

### Conflitos de merge no upstream
```bash
# Se houver conflitos durante o merge upstream
git status
# Editar arquivos com conflitos
git add .
git commit -m "Resolve upstream merge conflicts"
```

### Packagist não atualiza
- Verificar se a tag foi criada corretamente no GitHub
- Aguardar até 10 minutos para sincronização automática
- Tentar atualização manual novamente

### Problemas no composer update
```bash
# Limpar cache e tentar novamente
composer clear-cache
rm -rf vendor/
composer install
```

## 📝 Checklist de Validação Final

- [ ] Fork sincronizado com upstream Yii
- [ ] Branch de teste criada e validada
- [ ] Tag versionada criada e enviada
- [ ] Packagist atualizado e sincronizado
- [ ] Projeto legacy atualizado via PR
- [ ] Testes executados com sucesso
- [ ] Deploy realizado sem erros
- [ ] Monitoramento pós-deploy ativo

## 🏷️ Versionamento

Padrão de tags: `X.Y.Z-pN`
- X.Y.Z: Versão upstream do Yii
- pN: Patch interno da empresa (p1, p2, p3...)

**Exemplo**: `1.1.28-p1` = Yii 1.1.28 com patch 1 da empresa