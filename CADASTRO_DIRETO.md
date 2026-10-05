# Cadastro direto — preparado em 05/10/2026

Os botões da página de vendas apontam para `https://fvclares.github.io/cardapioonline/admin.html?signup=1`. O formulário de solicitação por e-mail foi removido. A conta é criada com e-mail e senha; quando exigido pelo Supabase, o usuário confirma seu e-mail antes de entrar. Após criar a loja, o teste segue a regra existente: até o primeiro dia do mês seguinte, no fuso de Fortaleza.

## Arquivos nos dois projetos

- `Site Cardápio/cardapio-venda/index.html`: links diretos, explicação do cadastro e prazo do teste.
- `Cardápios/admin.html`, `Cardápios/js/admin-supabase.js`, `Cardápios/js/lib/supabase.js`: cadastro público, tratamento de confirmação de e-mail, erros e conta já existente.
- `Cardápios/supabase/migrations/20261005142114_public_signup.sql`: permite cadastro sem convite; mantém validação de convites fornecidos e cria apenas o perfil owner.
- `Cardápios/tests/public-signup.test.js`, `Cardápios/tests/public-signup.sql`, `Cardápios/tests/run.mjs`: verificações do fluxo e do banco.

## Validação

Testes do sistema e build aprovados. A migração foi verificada no banco em transação com rollback: cadastro público, isolamento de perfil, convites válidos e inválidos, criação de loja e teste até o próximo dia 1. Nenhuma conta ou loja de teste ficou gravada.

## Publicação em 05/10/2026

A migração foi aplicada em produção e os testes de banco passaram novamente com rollback. O cadastro público está habilitado no Supabase Auth, com confirmação automática de e-mail. O sistema Cardápios foi publicado com o formulário em `/cardapioonline/admin.html?signup=1`. A publicação desta página de vendas ocorre pelo workflow GitHub Pages ao enviar este commit.

Os workflows usam nomes de artefatos diferentes por execução e tentativa, com uma breve espera antes da implantação, para evitar os erros de metadados ausentes e artefatos duplicados encontrados durante a publicação.

O arquivo `tests/public-signup.sql` deve ser executado apenas dentro de `BEGIN`/`ROLLBACK`. Não executar sem rollback.
