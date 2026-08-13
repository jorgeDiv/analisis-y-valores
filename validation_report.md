# Validación del Score — Dashboard de Inversión Digital

Recálculo independiente en Python sobre `cg50.json` (CoinGecko, 50 activos).
Filtro liquidez volumen 24h ≥ $10M → universo de **44** activos.
Sin valores nulos/NaN: la regla "nunca rellenar NaN con ceros" es inocua en este dataset.
El dashboard SÍ usa `pct(x)=x==null?0:x` (null→0), desviación de cumplimiento pero inerte aquí.

## Top 10 TAL COMO LO CALCULA EL DASHBOARD (score_final normalizado 0–100)

| # | símbolo | precio | momentum | vol | riesgo | score_asim* | score_final |
|---|---------|--------|----------|-----|--------|-------------|-------------|
| 1 | USDT  | 0.999  | 0.090 | 0.00 | 0.00 | 1.000 | 100.0 |
| 2 | USDC  | 0.999  | 0.090 | 0.00 | 0.00 | 1.000 | 95.1  |
| 3 | USD1  | 0.999  | 0.090 | 0.00 | 0.00 | 1.000 | 84.5  |
| 4 | USDG  | 1.000  | 0.090 | 0.00 | 0.00 | 1.000 | 80.9  |
| 5 | USDS  | 1.000  | 0.090 | 0.00 | 0.00 | 1.000 | 79.1  |
| 6 | DAI   | 1.000  | 0.090 | 0.00 | 0.00 | 1.000 | 78.5  |
| 7 | PYUSD | 1.000  | 0.090 | 0.00 | 0.00 | 1.000 | 78.0  |
| 8 | RLUSD | 1.000  | 0.090 | 0.00 | 0.00 | 1.000 | 77.4  |
| 9 | USDE  | 1.000  | 0.090 | 0.00 | 0.00 | 1.000 | 70.0  |
|10 | BTC   | 63380  | -0.153| 3.10 | 1.21 | 0.005 | 46.2  |

`* score_asim = asimetría normalizada 0–1 que usa el dashboard (asymNorm).`
`Sharpe-like crudo = momentum/(vol+0.001): 90.2 para todos los stablecoins; ~-1.2…+0.7 para el resto.`

## ⚠ Discrepancia crítica (defecto de diseño)
El top 10 lo dominan **stablecoins**. Causas:
1. `eps=0.001` hace que activos de volatilidad≈0 obtengan asimetría = momentum/0.001 ≈ 90,
   y riesgo≈0 (vol≈0 → riesgo≈0 → premio en (1−riesgo)).
2. Al min-max normalizar la asimetría sobre un universo con esos extremos (90), todos los
   cripto reales quedan comprimidos a asim_norm ≈ 0.005. El término de asimetría (50% del
   peso) aporta **≤0.38 de 100 puntos** → inerte. El score se decide por liquidez + bajo
   riesgo, NO por asimetría riesgo/beneficio como promete el dashboard.

## 2ª discrepancia (fórmula)
El dashboard normaliza la asimetría (asymNorm) antes de ponderar; el encargo indica
`score_final = 0.5*asim + ...` con `asim = momentum/(vol+eps)` (crudo, no acotado).
Ambas versiones producen el mismo top-10 de stablecoins; solo cambia BTC↔USDD en el corte.

## Top 10 corregido (ex-stablecoins, precio≈1 excluido)
BTC 46.2 · ETH 40.2 · SOL 35.1 · DOGE 31.0 · BNB 29.8 · TRX 28.9 · LTC 26.6 ·
HYPE 26.0 · SUI 25.7 · TAO 25.4. (Momentum mayormente negativo en este snapshot.)

## Metodología y limitaciones
momentum = z-score compuesto (pesos 0.4/0.3/0.2/0.1 en 24h/7d/30d/1h) sobre media y
desviación poblacional del universo filtrado; volatilidad proxy = |7d|+0.5|30d|;
riesgo(1–10)=10·vol/máx(vol); asimetría tipo-Sharpe = momentum/(vol+ε);
liquidez = log10(volumen) normalizado; score = 0.5·asim+0.3·liq+0.2·(1−riesgo).

Limitaciones:
- **Volatilidad proxy** solo usa 7d/30d en valor absoluto; ignora dispersión intradía y colas.
- **Sesgo de supervivencia**: muestra = 50 mayores por capitalización; excluye caídas y
  activos muertos, inflando la asimetría observada.
- `eps=0.001` genera asimetrías patológicas para vol≈0 (stablecoins).
- Snapshot único; sin serie temporal para z-scores robustos.

**Recomendación:** excluir stablecoins y elevar `eps` (volatilidad mínima real) antes de
producir el ranking.
