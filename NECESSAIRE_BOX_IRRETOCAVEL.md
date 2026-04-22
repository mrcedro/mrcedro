# Calculadora de Necessaire Box — Versão Irretocável (Especificação + Prompt)

## 1) Crítica técnica do que aconteceu até agora

1. **Inconsistência entre rótulo e fórmula do canal**
   - Em alguns trechos, o texto diz *"frente + verso"*, mas o cálculo foi corrigido para **perímetro de uma peça** (frente **ou** verso).
   - Isso confunde a costureira e gera corte errado.

2. **Regra de “caixinha de leite” sem governança clara**
   - A regra correta é: **só existe caixinha de leite com cantos retos**.
   - Se o app mostrar a seção com cantos arredondados, está incorreto.

3. **Dependência real da margem da máquina**
   - Você acertou em exigir campo digitável. Não pode ser inferido.
   - Falta tratar validações para não gerar resultado inválido (ex.: `cortar = margem_maquina - 0.5` ficar negativo).

4. **“Tipo de corte” sem impacto completo no fluxo**
   - A opção “um corte único” vs “frente e verso” precisa alterar:
     - instruções,
     - nomenclatura dos resultados,
     - e visualização.

5. **Visualização de corte precisa ser “fonte da verdade”**
   - O desenho precisa refletir 100% do modo selecionado (retos/arredondados + caixinha ligada/desligada).

6. **Defeito estrutural crítico no HTML/JS fornecido**
   - O JS busca `caixinha-section`, mas a seção no HTML não tem esse `id`.
   - Resultado: erro em runtime ao tentar acessar `style.display` de `null`.

---

## 2) Regras irretocáveis (negócio)

## Entradas obrigatórias
- Largura pronta (**L**)
- Altura pronta (**A**)
- Profundidade pronta (**P**)
- Margem de costura (**MA**)
- Margem da máquina (**MM**)
- Largura do zíper (**Z**)
- Tipo de corte: `unico` | `frente_verso`
- Tipo de canto: `retos` | `arredondados`
- Raio (**R**) obrigatório somente se `arredondados`

## Cálculos base
- `altura_corte = (2*A) + (2*P) + (2*MA)`
- `largura_corte = L + A + MA`
- `alca = 1.5 + L`

## Canal do zíper e fole
- **Comprimento do canal** = perímetro de **uma peça** (frente **ou** verso), não soma das duas.
- Se `retos`:
  - `perimetro = 2*(altura_corte + largura_corte)`
- Se `arredondados`:
  - `perimetro = 2*(altura_corte + largura_corte) - 8*R + 2*pi*R`
- **Largura do canal**:
  - `largura_canal = (2*P) + (2*MA) + Z`

## Caixinha de leite
- Só quando `tipo_canto = retos`.
- `costurar_no_risco = MM`
- `cortar_canto = max(MM - 0.5, 0)`  
  (nunca deixar negativo)

---

## 3) Regras de UX/UI (para não falhar na prática)

1. **Toggle explícito de canto**
   - Botões: `Retos` e `Arredondados`.
   - Se `Retos`: esconder input de raio e mostrar dica “cantos retos não usam raio”.
   - Se `Arredondados`: mostrar input de raio.

2. **Seção “Caixinha de leite” condicional**
   - Visível só em `Retos`.
   - Oculta em `Arredondados`.

3. **Visualização sincronizada**
   - `Retos`: retângulo com marcações de caixinha.
   - `Arredondados`: retângulo com `rx/ry` e indicação visual de raio.

4. **Texto sem ambiguidade**
   - Sempre mostrar: “Perímetro de uma peça (frente ou verso)”.
   - Nunca escrever “frente + verso” se a fórmula não somar as duas.

5. **Validação de entradas**
   - Não aceitar negativos.
   - Se `R > min(altura_corte, largura_corte)/2`, travar no máximo válido e avisar.

---

## 4) Prompt pronto para Canva AI (versão final)

> Crie/edite meu app “Calculadora Necessaire Box” com comportamento determinístico e sem inconsistências.
>
> **Objetivo**: calcular medidas de corte, canal do zíper/fole, caixinha de leite e alça para necessaire box.
>
> **Entradas obrigatórias (cm)**:
> - Largura pronta (L)
> - Altura pronta (A)
> - Profundidade pronta (P)
> - Margem de costura (MA)
> - Margem da máquina (MM) **digitável**
> - Largura do zíper (Z) **digitável**
> - Tipo de corte: “Um corte único” ou “Frente e verso”
> - Tipo de canto: “Retos” ou “Arredondados”
> - Raio (R): aparece só quando “Arredondados”
>
> **Cálculos**:
> - altura_corte = (2*A) + (2*P) + (2*MA)
> - largura_corte = L + A + MA
> - alca = 1.5 + L
> - largura_canal = (2*P) + (2*MA) + Z
> - perimetro (canal) = perímetro de **uma peça** (frente OU verso)
>   - cantos retos: 2*(altura_corte + largura_corte)
>   - cantos arredondados: 2*(altura_corte + largura_corte) - 8*R + 2*pi*R
>
> **Regra crítica da caixinha de leite**:
> - Mostrar e calcular **somente** se canto = Retos.
> - costurar_no_risco = MM
> - cortar_canto = max(MM - 0.5, 0)
>
> **Regras de interface**:
> - O toggle de cantos deve mudar cálculos e visualização do corte em tempo real.
> - Se Retos: ocultar raio, exibir caixinha e linhas de caixinha no diagrama.
> - Se Arredondados: exibir raio, ocultar caixinha.
> - Seção “Canal do zíper e fole” deve mostrar texto: “Perímetro de uma peça (frente ou verso)”.
>
> **Correção obrigatória de código**:
> - Garanta que a seção de caixinha tenha `id="caixinha-section"` e que o JS não quebre ao alternar cantos.
>
> **Saída desejada**:
> - App sem erros de runtime.
> - Cálculos coerentes com os rótulos.
> - Visualização fiel ao tipo de canto.

---

## 5) Critérios de aceite (checklist)

- [ ] Campo “Margem da máquina” digitável e usado no cálculo.
- [ ] Campo “Largura do zíper” digitável e usado no cálculo.
- [ ] “Canal” usa perímetro de uma peça (não dobro).
- [ ] Caixinha de leite só aparece em cantos retos.
- [ ] Visualização muda corretamente entre retos/arredondados.
- [ ] Sem erro no console ao alternar opções.
- [ ] Textos/fórmulas exibidas condizem com o valor calculado.
