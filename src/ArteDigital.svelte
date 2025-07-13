<script>
    import './ArteDigital.css';
    import marcoTopHombre from '/public/images/top hombre.png';
    import marcoTopMujer  from '/public/images/top mujer.png';
    import marcoTopOtro   from '/public/images/top otro.png';
    import marcoBottom    from '/public/images/bottom.png';
    import tornillo       from '/public/images/tornillo.png';

    export let distribucion = [];  // array de 50 colores hex
    export let nombre = '';
    export let rareza = '';
    export let sexo = '';          // prop para controlar el marco superior
    export let viveSolo = '';      // prop para controlar si mostrar tornillos

    // Selecciona la imagen y clase del marco superior según sexo
    $: topFrameSrc = sexo === 'femenino'
    ? marcoTopMujer
    : sexo === 'masculino'
        ? marcoTopHombre
        : marcoTopOtro;
    $: sexoClass = sexo === 'femenino'
    ? 'mujer'
    : sexo === 'masculino'
        ? 'hombre'
        : 'otro';
</script>

<div class="arte-container">
  <!-- Capa 1: Marco superior dinámico -->
  <img
    src={topFrameSrc}
    alt="Marco superior"
    class="frame-top {sexoClass}"
  />

  <!-- Capa 2: Marco inferior fijo -->
  <img src={marcoBottom} alt="Marco inferior" class="frame-bottom" />

  <!-- Capa 3: Tornillos solo si vive solo -->
  {#if viveSolo === 'sí'}
    <img src={tornillo} alt="Tornillo izquierdo" class="tornillo left" />
    <img src={tornillo} alt="Tornillo derecho" class="tornillo right" />
  {/if}

  <!-- Capa 4: Rejilla de cuadrados -->
  <div class="grid-cuadrados">
    {#each distribucion as color}
      <div class="cuadrado" style="background-color: {color}"></div>
    {/each}
  </div>

  <!-- Capa 5: Info -->
  <div class="info-arte">
    <span class="nombre">{nombre}</span>
    <span class="rareza">{rareza}</span>
  </div>
</div>
