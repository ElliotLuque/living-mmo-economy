# Living MMO Economy — YAML Configuration Documentation

Esta documentación describe el contrato de configuración YAML para la simulación **Living MMO Economy**.

Los YAML no representan estado vivo, sino una **WorldDefinition**: una descripción inicial del mundo que luego el motor valida y convierte en `WorldState`.

```text
YAML files
  -> WorldDefinition
  -> Validation
  -> WorldBuilder
  -> WorldState
  -> SimulationEngine
```

---

## Índice

1. [`world.yml`](#worldyml)
2. [`economy.yml`](#economyyml)
3. [`item-taxonomy.yml`](#item-taxonomyyml)
4. [`items.yml`](#itemsyml)
5. [`professions.yml`](#professionsyml)
6. [`regions.yml`](#regionsyml)
7. [`recipes.yml`](#recipesyml)
8. [`vendors.yml`](#vendorsyml)
9. [`agents.yml`](#agentsyml)
10. [`spawns.yml`](#spawnsyml)
11. [Reglas globales del motor](#reglas-globales-del-motor)

---

# `world.yml`

Define la identidad del mundo y parámetros generales de simulación.

## Ejemplo

```yaml
world:
  id: default
  name: "Default MMO Economy"
  description: "Small MMO Economy simulation focused on Herbalism, Alchemy and raid demand"

simulation:
  initialTick: 0
  tickDurationMinutes: 5
  randomSeed: 12345
  snapshotEveryTicks: 15
  maxTimelineEvents: 100
```

## Campos

### `world.id`

Identificador único del mundo.

Debe ser estable, porque puede usarse para cargar presets, snapshots o simulation runs.

### `world.name`

Nombre legible del mundo.

### `world.description`

Descripción del escenario económico.

### `simulation.initialTick`

Tick inicial de la simulación.

Normalmente:

```yaml
initialTick: 0
```

### `simulation.tickDurationMinutes`

Cuántos minutos simulados representa un tick.

Ejemplo:

```yaml
tickDurationMinutes: 5
```

Esto afecta a cómo interpretar:

```text
cycleTicks
craftingTimeTicks
durationTicks
listingDurationTicks
regenPerTick
```

### `simulation.randomSeed`

Seed usada para generar resultados reproducibles.

Si ejecutas la misma simulación con los mismos datos y seed, deberías poder reproducir resultados.

### `simulation.snapshotEveryTicks`

Cada cuántos ticks se genera un snapshot para consola/API/frontend.

### `simulation.maxTimelineEvents`

Número máximo de eventos recientes conservados en timeline.

## Validaciones

```text
world.id no vacío
simulation.tickDurationMinutes > 0
simulation.snapshotEveryTicks > 0
simulation.maxTimelineEvents > 0
```

---

# `economy.yml`

Define reglas económicas globales.

## Ejemplo

```yaml
economy:
  currency: GOLD

  market:
    defaultListingDurationTicks: 48
    listingFeeRate: 0.01
    transactionFeeRate: 0.05

  pricing:
    priceMemoryWindowTicks: 100
    minListingPrice: 0.01
```

## Campos

### `economy.currency`

Moneda base del mundo.

Ejemplo:

```yaml
currency: GOLD
```

Todos los precios, recompensas, compras, ventas, fees e ingresos se expresan en esta moneda.

### `market.defaultListingDurationTicks`

Duración de un listing antes de expirar.

Si un listing expira:

```text
la cantidad restante vuelve al vendedor
el vendedor aplica reaction.expired
```

### `market.listingFeeRate`

Fee cobrada al publicar un listing.

Ejemplo:

```yaml
listingFeeRate: 0.01
```

Significa 1%.

Esta fee es un sumidero de oro.

### `market.transactionFeeRate`

Fee cobrada cuando ocurre una venta.

Ejemplo:

```yaml
transactionFeeRate: 0.05
```

Si un item se vende por 100g:

```text
buyer paga 100g
seller recibe 95g
5g desaparecen como fee
```

### `pricing.priceMemoryWindowTicks`

Ventana temporal usada para calcular memoria de precios.

Ejemplo:

```yaml
priceMemoryWindowTicks: 100
```

El mercado puede calcular:

```text
último precio vendido
precio medio reciente
volumen reciente
volatilidad
```

sobre las transacciones ocurridas en esta ventana.

### `pricing.minListingPrice`

Precio mínimo permitido para un listing.

Evita precios cero, negativos o absurdos.

## Validaciones

```text
defaultListingDurationTicks > 0
0 <= listingFeeRate <= 1
0 <= transactionFeeRate <= 1
priceMemoryWindowTicks > 0
minListingPrice > 0
```

---

# `item-taxonomy.yml`

Define la taxonomía válida de items.

La taxonomía controla qué combinaciones `categoryId + typeId` son válidas.

## Ejemplo

```yaml
itemTaxonomy:
  categories:
    - id: MATERIAL
      name: "Material"
      types:
        - id: HERB
          name: "Herb"
        - id: VENDOR_SUPPLY
          name: "Vendor supply"

    - id: CONSUMABLE
      name: "Consumable"
      types:
        - id: POTION
          name: "Potion"
        - id: FLASK
          name: "Flask"
```

## Campos

### `categories[].id`

Familia grande del item.

Ejemplos:

```text
MATERIAL
CONSUMABLE
EQUIPMENT
COSMETIC
```

### `categories[].types[].id`

Subtipo dentro de la categoría.

Ejemplos:

```text
MATERIAL/HERB
MATERIAL/VENDOR_SUPPLY
CONSUMABLE/POTION
CONSUMABLE/FLASK
```

## Para qué sirve

Permite validar items como:

```yaml
categoryId: CONSUMABLE
typeId: POTION
```

Y rechazar combinaciones inválidas como:

```yaml
categoryId: MATERIAL
typeId: POTION
```

También sirve para demandas de consumers:

```yaml
target:
  categoryId: CONSUMABLE
  typeId: POTION
```

## Validaciones

```text
category.id único
type.id único dentro de cada category
todo item debe usar una combinación válida categoryId/typeId
todo demand target debe usar una combinación válida categoryId/typeId
```

---

# `items.yml`

Define los items existentes en el mundo.

Los items no tienen `basePrice`. El precio emerge del mercado.

## Ejemplo

```yaml
items:
  - id: silverthorn
    name: "Silverthorn"
    categoryId: MATERIAL
    typeId: HERB
    rarity: COMMON
    stackable: true
    gathering:
      professionId: HERBALISM
      requiredLevel: 1

  - id: crystal_vial
    name: "Crystal vial"
    categoryId: MATERIAL
    typeId: VENDOR_SUPPLY
    rarity: COMMON
    stackable: true

  - id: minor_healing_potion
    name: "Minor healing potion"
    categoryId: CONSUMABLE
    typeId: POTION
    rarity: COMMON
    stackable: true
```

## Campos

### `id`

Identificador único del item.

Se referencia desde:

```text
recipes.yml
regions.yml
vendors.yml
market listings
inventories
transactions
```

### `name`

Nombre legible.

### `categoryId`

Categoría principal del item.

Debe existir en `item-taxonomy.yml`.

### `typeId`

Tipo dentro de la categoría.

Debe existir dentro de la categoría declarada.

### `rarity`

Rareza del item.

Ejemplos:

```text
COMMON
UNCOMMON
RARE
EPIC
LEGENDARY
```

De momento puede ser descriptivo. Más adelante podría afectar a demanda, drop, XP o rareza de nodos.

### `stackable`

Indica si el item puede acumularse en inventario.

Para el MVP, casi todo puede ser `true`.

### `gathering`

Solo aparece en items recolectables.

Ejemplo:

```yaml
gathering:
  professionId: HERBALISM
  requiredLevel: 15
```

Indica que para recolectar ese item hace falta una profesión concreta y nivel mínimo.

Los items crafteados o de vendor no tienen `gathering`.

## Validaciones

```text
item.id único
categoryId/typeId válido en taxonomy
rarity válida
si gathering existe:
  professionId existe
  requiredLevel >= 1
  requiredLevel <= profession.maxLevel
```

---

# `professions.yml`

Define profesiones y progresión.

## Ejemplo

```yaml
professions:
  - id: HERBALISM
    name: "Herbalism"
    type: GATHERING
    maxLevel: 100
    progression:
      xpPerLevel: 100

  - id: ALCHEMY
    name: "Alchemy"
    type: CRAFTING
    maxLevel: 100
    progression:
      xpPerLevel: 100
```

## Campos

### `id`

Identificador de la profesión.

Referenciado por:

```text
items.yml gathering.professionId
recipes.yml professionId
agents.yml behavior.professions[].id
```

### `type`

Tipo de profesión.

Valores actuales:

```text
GATHERING
CRAFTING
```

### `maxLevel`

Nivel máximo de la profesión.

### `progression.xpPerLevel`

XP necesaria para subir un nivel.

Modelo actual:

```text
LINEAR
```

Si `xpPerLevel = 100`, cada 100 XP se sube 1 nivel.

Se asume `startingXp = 0` siempre.

## Validaciones

```text
profession.id único
maxLevel > 0
xpPerLevel > 0
```

---

# `regions.yml`

Define regiones y nodos de recursos.

## Ejemplo

```yaml
regions:
  - id: greenwood
    name: "Greenwood forest"
    resourceNodes:
      - id: greenwood_silverthorn_patch
        itemId: silverthorn
        initialStock: 250
        maxStock: 600
        regenPerTick: 3.0
        gather:
          baseAmount:
            min: 2
            max: 4
          maxAmount: 7
          skillScaling:
            levelsPerBonusUnit: 12
          professionXp:
            base: 5
```

## Campos

### `regions[].id`

Identificador de región.

### `resourceNodes[].id`

Identificador único del nodo.

### `resourceNodes[].itemId`

Item producido por el nodo.

Debe existir en `items.yml`.

Ese item debe tener `gathering`.

### `initialStock`

Stock inicial del nodo al arrancar el mundo.

### `maxStock`

Stock máximo acumulable.

La regeneración nunca puede superar este límite.

### `regenPerTick`

Cantidad de stock que se regenera por tick.

Puede ser decimal.

Si es decimal, el motor debe usar acumulador fraccional.

Ejemplo:

```text
regenPerTick = 0.4

tick 1: acumula 0.4
tick 2: acumula 0.8
tick 3: acumula 1.2 -> añade 1 stock, conserva 0.2
```

### `gather.baseAmount.min/max`

Cantidad base que un agente puede recolectar en una acción.

Ejemplo:

```text
baseAmount: 2-4
```

### `gather.maxAmount`

Cantidad máxima recolectable por acción tras aplicar bonus de nivel.

### `gather.skillScaling.levelsPerBonusUnit`

Cada cuántos niveles por encima del requisito se obtiene +1 unidad recolectada.

Fórmula:

```text
bonus = floor((professionLevel - requiredLevel) / levelsPerBonusUnit)

amount = random(baseMin, baseMax) + bonus

amount = min(amount, maxAmount)
amount = min(amount, currentNodeStock)
```

### `gather.professionXp.base`

XP ganada al recolectar con éxito.

Si el agente recolecta 0, no gana XP.

## Validaciones

```text
region.id único
node.id único
itemId existe
itemId es recolectable
initialStock >= 0
maxStock > 0
initialStock <= maxStock
regenPerTick >= 0
baseAmount.min > 0
baseAmount.max >= baseAmount.min
maxAmount >= baseAmount.max
levelsPerBonusUnit > 0
professionXp.base > 0
```

---

# `recipes.yml`

Define recetas de crafting.

## Ejemplo

```yaml
recipes:
  - id: minor_healing_potion_recipe
    name: "Minor healing potion"
    professionId: ALCHEMY
    requiredLevel: 1
    craftingTimeTicks: 3
    professionXp:
      base: 10
    output:
      itemId: minor_healing_potion
      quantity: 1
    ingredients:
      - itemId: silverthorn
        quantity: 3
      - itemId: crystal_vial
        quantity: 1
```

## Campos

### `id`

Identificador único de receta.

### `professionId`

Profesión necesaria para fabricar.

Debe existir en `professions.yml` y ser de tipo `CRAFTING`.

### `requiredLevel`

Nivel mínimo de profesión.

### `craftingTimeTicks`

Duración del crafting.

Mientras craftea, el agente queda ocupado.

Para MVP:

```text
un crafter solo puede tener un crafting job activo
```

### `professionXp.base`

XP ganada al completar la receta.

### `output.itemId`

Item producido.

Debe existir en `items.yml`.

### `output.quantity`

Cantidad producida.

### `ingredients`

Lista de ingredientes.

Cada `itemId` debe existir.

Regla del motor:

```text
el crafter compra ingredientes all-or-nothing
si no puede comprar todos, no compra nada
```

## Validaciones

```text
recipe.id único
professionId existe
profession type = CRAFTING
requiredLevel >= 1
requiredLevel <= profession.maxLevel
craftingTimeTicks > 0
professionXp.base > 0
output.itemId existe
output.quantity > 0
ingredients no vacío
ingredient.itemId existe
ingredient.quantity > 0
```

---

# `vendors.yml`

Define vendedores NPC o supplies infinitos/controlados.

Se usa para materiales auxiliares que no quieres simular como economía emergente.

## Ejemplo

```yaml
vendors:
  - id: alchemy_supplier
    name: "Alchemy Supplier"
    listings:
      - itemId: crystal_vial
        price: 2.00
        stockMode: INFINITE
```

## Campos

### `id`

Identificador del vendor.

### `listings[].itemId`

Item vendido por el vendor.

Debe existir en `items.yml`.

Normalmente será:

```text
MATERIAL/VENDOR_SUPPLY
```

### `price`

Precio fijo del item.

El oro pagado al vendor sale de la economía.

### `stockMode`

Modo de stock.

Valores recomendados:

```text
INFINITE
LIMITED
```

Para MVP, `INFINITE` basta.

## Para qué sirve

Ejemplo:

```text
Alchemy recipe necesita crystal_vial.
No quieres simular producción de viales.
Vendor lo vende infinito a 2g.
```

Esto evita que el crafter se bloquee y crea un sumidero de oro.

## Validaciones

```text
vendor.id único
vendor listing itemId existe
price > 0
stockMode válido
todo VENDOR_SUPPLY usado en recetas debe estar disponible en vendor o supply equivalente
```

---

# `agents.yml`

Define arquetipos de agentes. No define individuos vivos.

Los individuos se crean desde `spawns.yml`.

---

## Farmer

Ejemplo:

```yaml
- id: basic_gatherer
  type: FARMER
  nameTemplate: "Basic Gatherer"
  behavior:
    strategy: SIMPLE_GATHER_AND_SELL
    actionCooldownTicks: 2
    professions:
      - id: HERBALISM
        startingLevel: 5
    gathering:
      nodeSelection: HIGHEST_AVAILABLE_STOCK
    pricing:
      initialAsk:
        mode: RANDOM_RANGE
        min: 2.00
        max: 8.00
      reaction:
        fastSoldOut:
          withinTicks: 2
          multiplier: 1.30
        soldOut:
          withinTicks: 8
          multiplier: 1.15
        expired:
          multiplier: 0.85
```

### `type: FARMER`

Agente que recolecta recursos y los lista en el mercado.

### `strategy: SIMPLE_GATHER_AND_SELL`

Comportamiento:

```text
buscar nodo recolectable
recolectar
ganar XP
listar stock
ajustar precio según venta/expiración
```

### `actionCooldownTicks`

Cada cuántos ticks puede actuar.

### `professions`

Profesiones iniciales del arquetipo.

```yaml
- id: HERBALISM
  startingLevel: 5
```

Se asume `startingXp = 0`.

### `gathering.nodeSelection`

Estrategia de selección de nodo.

Valores actuales:

```text
HIGHEST_AVAILABLE_STOCK
WEIGHTED_BY_AVAILABLE_STOCK
```

`HIGHEST_AVAILABLE_STOCK`:

```text
elige el nodo recolectable con más stock
```

`WEIGHTED_BY_AVAILABLE_STOCK`:

```text
elige entre nodos recolectables con probabilidad proporcional al stock disponible
```

### `pricing.initialAsk`

Precio inicial cuando el agente no tiene memoria para ese item.

```yaml
mode: RANDOM_RANGE
min: 2.00
max: 8.00
```

### `pricing.reaction.fastSoldOut`

Si el listing se vende muy rápido, el agente sube fuerte su expectativa.

### `pricing.reaction.soldOut`

Si el listing se vende dentro de una ventana razonable, sube moderadamente.

### `pricing.reaction.expired`

Si el listing expira sin venderse, baja su expectativa.

---

## Crafter

Ejemplo:

```yaml
- id: alchemist
  type: CRAFTER
  nameTemplate: "Alchemist"
  behavior:
    strategy: PROFIT_BASED_CRAFTING
    actionCooldownTicks: 1
    professions:
      - id: ALCHEMY
        startingLevel: 25
    decision:
      minRoi: 0.20
      maxGoldInvestmentRatio: 0.60
      maxUnsoldOutputStock: 5
    exploration:
      allowUnknownMarketCrafting: true
      maxExplorationInvestmentRatio: 0.15
      maxExplorationBatchSize: 1
    pricing:
      fallbackOutputAsk:
        mode: COST_PLUS_RANDOM_MARKUP
        minMarkup: 1.40
        maxMarkup: 2.50
      reaction:
        fastSoldOut:
          withinTicks: 2
          multiplier: 1.25
        soldOut:
          withinTicks: 8
          multiplier: 1.10
        expired:
          multiplier: 0.90
```

### `strategy: PROFIT_BASED_CRAFTING`

Elige recetas por rentabilidad esperada.

Orden conceptual:

```text
1. filtra recetas por profesión/nivel
2. descarta outputs con demasiado stock sin vender
3. estima coste de ingredientes
4. estima precio de venta del output
5. calcula ROI
6. elige la receta con mayor ROI
7. si empate, orden determinista
```

### `decision.minRoi`

ROI mínimo para fabricar.

```text
ROI = beneficio esperado / coste
```

### `decision.maxGoldInvestmentRatio`

Máximo porcentaje de oro que puede invertir en una operación.

### `decision.maxUnsoldOutputStock`

Máximo stock de output sin vender.

Debe contar:

```text
inventario del agente + listings activos del agente
```

### `exploration.allowUnknownMarketCrafting`

Permite fabricar productos sin histórico de mercado.

### `exploration.maxExplorationInvestmentRatio`

Máximo porcentaje de oro que puede arriesgar en exploración.

### `exploration.maxExplorationBatchSize`

Cantidad máxima a fabricar en una prueba.

### `pricing.fallbackOutputAsk`

Precio usado solo si no hay información fiable de mercado.

El orden recomendado de pricing es:

```text
1. transacciones recientes
2. listings actuales
3. memoria propia
4. fallbackOutputAsk
```

---

## Consumer

Ejemplo:

```yaml
- id: casual_pve
  type: CONSUMER
  nameTemplate: "Casual PvE Consumer"
  behavior:
    strategy: NEED_BASED_CONSUMPTION
    actionCooldownTicks: 1
    income:
      mode: NEED_BASED
      source: QUESTING
      work:
        trigger: INSUFFICIENT_GOLD_FOR_DEMAND
        durationTicks:
          min: 3
          max: 8
        maxAttemptsPerDemandCycle: 3
        rewardPerTick:
          min: 8.00
          max: 20.00
    demands:
      - id: basic_potions
        target:
          categoryId: CONSUMABLE
          typeId: POTION
        quantityPerCycle: 3
        cycleTicks: 24
        priority: HIGH
        urgency:
          normalMultiplier: 1.00
          unsatisfiedDemandMultiplier: 1.10
          maxMultiplier: 2.50
        consumeImmediately: true
```

### `strategy: NEED_BASED_CONSUMPTION`

El consumer tiene demandas periódicas.

Si tiene oro suficiente:

```text
compra y consume
```

Si no tiene oro suficiente:

```text
inicia work externo
espera durationTicks
recibe reward
reevalúa el mercado
```

### `income.mode: NEED_BASED`

No hay income automático cada X ticks.

El consumer solo trabaja cuando necesita oro para cubrir una demanda.

### `work.trigger`

Actualmente:

```text
INSUFFICIENT_GOLD_FOR_DEMAND
```

El work se dispara si el consumer no tiene oro suficiente para cubrir su demanda.

### `work.durationTicks`

Duración del trabajo externo.

Durante este tiempo, el consumer está ocupado.

### `work.maxAttemptsPerDemandCycle`

Número máximo de trabajos externos por ciclo de demanda.

El número real de attempts puede depender de `urgency`.

Regla recomendada:

```text
urgency < 1.25  -> 1 attempt
urgency < 1.75  -> 2 attempts
urgency >= 1.75 -> 3 attempts

attempts = min(attempts, maxAttemptsPerDemandCycle)
```

### `work.rewardPerTick`

Recompensa generada por tick de work.

```text
reward = durationTicks * random(rewardPerTick.min, rewardPerTick.max)
```

### `demands[].quantityPerCycle`

Cantidad deseada por ciclo.

### `demands[].cycleTicks`

Cada cuántos ticks aparece la demanda.

### `demands[].urgency`

Controla cuánto insiste el consumer en cubrir esa demanda.

No representa budget.

Afecta a:

```text
número de work attempts
prioridad entre demandas
insistencia tras demanda insatisfecha
```

### `consumeImmediately`

Si es `true`, el item desaparece al comprarlo.

## Validaciones de agents

```text
agent.id único
type válido
strategy compatible con type
profession existe
startingLevel válido
actionCooldownTicks > 0
initialAsk min <= max
pricing multipliers coherentes
fastSoldOut.withinTicks < soldOut.withinTicks
expired.multiplier < 1
minRoi >= 0
0 < maxGoldInvestmentRatio <= 1
0 < maxExplorationInvestmentRatio <= 1
maxExplorationBatchSize > 0
durationTicks.min <= durationTicks.max
rewardPerTick.min <= rewardPerTick.max
demand target existe en taxonomy
quantityPerCycle > 0
cycleTicks > 0
urgency maxMultiplier >= normalMultiplier
```

---

# `spawns.yml`

Define cuántos agentes runtime se crean desde cada arquetipo.

## Ejemplo

```yaml
spawns:
  - agentId: basic_gatherer
    count: 4
    startingGold:
      min: 5.00
      max: 15.00

  - agentId: alchemist
    count: 3
    startingGold:
      min: 80.00
      max: 180.00
```

## Campos

### `agentId`

Referencia a un arquetipo definido en `agents.yml`.

### `count`

Número de agentes runtime creados.

### `startingGold`

Oro inicial aleatorio.

## Validaciones

```text
agentId existe
count > 0
startingGold.min >= 0
startingGold.max >= startingGold.min
```

---

# Reglas globales del motor

Estas reglas no tienen por qué estar en YAML.

## Crafter all-or-nothing

El crafter nunca compra ingredientes parcialmente.

```text
si puede comprar todos los ingredientes:
  compra
si falta algo:
  no compra nada
```

## Consumer partial satisfaction

El consumer sí puede satisfacer parcialmente una demanda.

Ejemplo:

```text
quiere 3 potions
solo puede comprar 1
compra 1
quedan 2 insatisfechas
```

## Price emergence

No existe `basePrice`.

El precio nace de:

```text
listings
transacciones
memoria individual
memoria de mercado
fallback strategies
```

## Listing expiration

Todo listing expira tras:

```yaml
economy.market.defaultListingDurationTicks
```

Al expirar:

```text
stock restante vuelve al vendedor
el vendedor aplica pricing.reaction.expired
```

## Regen fraccional

Si `regenPerTick` es decimal, el nodo debe conservar acumulador interno.

---
