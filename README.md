# Armazenamento de Dados no Navegador com React

Existem várias formas de armazenar dados no navegador quando se trabalha com React. Cada uma tem suas particularidades, vantagens e casos de uso ideais. Vamos explorar as principais opções:
## Sumário
1. LocalStorage
2. SessionStorage
3. Cookies
4. IndexedDB
5. Cache API
6. React Context + Estado
7. Bibliotecas de Gerenciamento de Estado

- Melhores Práticas
- Comparação de Opções

---

## 1. LocalStorage

**Características**:
- Armazenamento persistente (dados permanecem após fechar o navegador)
- Capacidade de ~5MB
- Apenas armazena strings (precisa serializar objetos com `JSON.stringify()`)
- Acesso síncrono

**Exemplo de uso**:
```javascript
// Salvar dados
localStorage.setItem('user', JSON.stringify({ name: 'João', id: 123 }));

// Recuperar dados
const userData = JSON.parse(localStorage.getItem('user') || '{}');

// Remover dados
localStorage.removeItem('user');

// Limpar tudo
localStorage.clear();
```

**Quando usar**: Para dados que precisam persistir entre sessões e não são sensíveis (como preferências do usuário, token de autenticação).

---

## 2. SessionStorage

**Características**:
- Similar ao LocalStorage, mas os dados são apagados quando a sessão do navegador termina (quando a aba é fechada)
- Capacidade de ~5MB
- Acesso síncrono

**Exemplo de uso**:
```javascript
// Salvar dados
sessionStorage.setItem('tempData', JSON.stringify({ cartItems: [...] }));

// Recuperar dados
const tempData = JSON.parse(sessionStorage.getItem('tempData') || '{}');
```

**Quando usar**: Para dados temporários que só são necessários durante uma sessão específica (como itens no carrinho de compras).

---

## 3. Cookies

**Características**:
- Pequena capacidade (~4KB por cookie)
- Podem ter data de expiração
- São enviados automaticamente em requisições HTTP
- Podem ser configurados como HTTP-only (inacessível via JavaScript) para segurança

**Exemplo de uso (com js-cookie)**:
```javascript
import Cookies from 'js-cookie';

// Salvar cookie
Cookies.set('token', 'abc123', { expires: 7 }); // Expira em 7 dias

// Recuperar cookie
const token = Cookies.get('token');

// Remover cookie
Cookies.remove('token');
```

**Quando usar**: Para tokens de autenticação (especialmente se marcados como HTTP-only e Secure) ou quando você precisa que o servidor tenha acesso aos dados.

---

## 4. IndexedDB

**Características**:
- Banco de dados NoSQL no navegador
- Grande capacidade (geralmente 50% do espaço livre no disco)
- Operações assíncronas
- Suporte a transações e índices

**Exemplo de uso (com Dexie.js)**:
```javascript
import Dexie from 'dexie';

const db = new Dexie('MyDatabase');
db.version(1).stores({
  friends: '++id, name, age'
});

// Inserir dados
await db.friends.add({ name: 'Maria', age: 25 });

// Consultar dados
const youngFriends = await db.friends.where('age').below(30).toArray();
```

**Quando usar**: Para aplicações complexas que precisam armazenar grandes quantidades de dados estruturados offline (como aplicações PWA).

---

## 5. Cache API

**Características**:
- Parte da especificação Service Workers
- Armazena respostas de requisições HTTP
- Útil para estratégias de cache offline

**Exemplo de uso**:
```javascript
caches.open('my-cache').then(cache => {
  cache.add('/api/data').then(() => {
    console.log('Dados armazenados no cache');
  });
});

// Recuperar depois
caches.match('/api/data').then(response => {
  if (response) {
    return response.json();
  }
});
```

**Quando usar**: Para recursos estáticos ou respostas de API que você quer disponíveis offline.

---

## 6. React Context + Estado

**Características**:
- Não é persistente (dados são perdidos ao atualizar a página)
- Acesso imediato em toda a aplicação
- Ideal para estado global da aplicação

**Exemplo de uso**:
```javascript
import { createContext, useContext, useState } from 'react';

const UserContext = createContext();

function App() {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {/* Componentes filhos */}
    </UserContext.Provider>
  );
}

function ChildComponent() {
  const { user } = useContext(UserContext);
  // ...
}
```

**Quando usar**: Para estado da aplicação que precisa ser acessado por muitos componentes, mas não precisa persistir.

---

## 7. Bibliotecas de Gerenciamento de Estado

**Exemplos**:
- Redux (com Redux Persist para persistência)
- Zustand
- MobX
- Recoil

**Exemplo com Zustand + persistência**:
```javascript
import create from 'zustand';
import { persist } from 'zustand/middleware';

const useStore = create(persist(
  (set) => ({
    count: 0,
    increment: () => set(state => ({ count: state.count + 1 })),
  }),
  {
    name: 'app-storage', // nome no localStorage
  }
));

// No componente
function Counter() {
  const { count, increment } = useStore();
  return <button onClick={increment}>{count}</button>;
}
```

---

## Comparação de Opções

| Método           | Persistência | Capacidade | Acesso  | Complexidade | Caso de Uso Típico |
|------------------|--------------|------------|---------|--------------|--------------------|
| LocalStorage     | Persistente  | ~5MB       | Síncrono | Baixa        | Preferências, tokens |
| SessionStorage   | Sessão       | ~5MB       | Síncrono | Baixa        | Dados temporários |
| Cookies          | Configurável | ~4KB       | Síncrono | Média        | Autenticação |
| IndexedDB        | Persistente  | Grande     | Assínc. | Alta         | Apps complexas offline |
| Cache API        | Persistente  | Variável   | Assínc. | Média        | Recursos offline |
| React Context    | Não persiste | -          | Imediato | Baixa        | Estado global |
| Redux/Zustand    | Opcional     | -          | Imediato | Média-Alta   | Estado complexo |

---

## Melhores Práticas

1. **Segurança**: Nunca armazene dados sensíveis (como senhas) no armazenamento do cliente sem criptografia.

2. **Serialização**: Para LocalStorage/SessionStorage, lembre-se de serializar objetos com `JSON.stringify()`.

3. **Quotas**: Monitore o uso de espaço para evitar erros de quota excedida.

4. **Fallbacks**: Sempre trate casos onde o armazenamento pode não estar disponível (modo privado, bloqueio de cookies).

5. **Sincronização**: Se trabalhar com dados remotos, implemente estratégias para sincronizar quando a conexão voltar.

6. **Limpeza**: Remova dados desnecessários para evitar acumulação.

A escolha do método ideal depende dos requisitos específicos do seu aplicativo em termos de persistência, tamanho dos dados, necessidade de acesso offline e requisitos de segurança.
