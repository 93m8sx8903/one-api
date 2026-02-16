# Usando a API para Controlar e Estender o One API

> PRs são bem-vindos para adicionar seus projetos de extensão aqui.

Por exemplo, embora o One API não tenha suporte direto a pagamentos, você pode implementar funcionalidade de pagamento através da API de extensão do sistema.

Ou, se quiser personalizar a estratégia de gerenciamento de canais, também pode usar a API para habilitar e desabilitar canais.

---

## Autenticação

O One API suporta dois métodos de autenticação: **Cookie** e **Token**.

Para Token, obtenha-o através do painel de administração (seção de tokens do sistema).

Depois, use o Token como valor do campo `Authorization` no header da requisição. Por exemplo, usando o Token para chamar a API de teste de canal:

```
Authorization: Bearer SEU_TOKEN_AQUI
```

---

## Formato de Requisição e Resposta

O One API usa formato **JSON** para requisições e respostas.

Para o corpo da resposta, o formato geral é:

```json
{
  "message": "Mensagem da requisição",
  "success": true,
  "data": {}
}
```

---

## Lista de APIs

> A lista atual de APIs não está completa. Capture as requisições do frontend pelo navegador para ver todas as disponíveis.

Se as APIs existentes não atenderem às suas necessidades, abra uma issue para discussão.

### Obter informações do usuário logado

**GET** `/api/user/self`

### Recarregar cota para um usuário específico

**POST** `/api/topup`

```json
{
  "user_id": 1,
  "quota": 100000,
  "remark": "Recarga de 100000 de cota"
}
```

---

## Outros

### Parâmetros adicionais no link de recarga

O One API anexa as informações do usuário e da recarga ao link quando o usuário clica no botão de recarga. Por exemplo:

```
https://example.com?username=root&user_id=1&transaction_id=4b3eed80-55d5-443f-bd44-fb18c648c837
```

Você pode parsear os parâmetros do link para obter as informações do usuário e da recarga, e então chamar a API para recarregar a cota do usuário.

**Nota:** Nem todos os temas suportam essa funcionalidade. PRs são bem-vindos para completar o suporte.
