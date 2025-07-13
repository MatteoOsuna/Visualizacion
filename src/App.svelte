<script>
  import './app.css';
  import AOS from 'aos';
  import 'aos/dist/aos.css';
  import ArteDigital from './ArteDigital.svelte';
  import { onMount } from 'svelte';
  import html2canvas from 'html2canvas';

  const nfts = [
    {
      id: 1,
      name: 'Monkey Ape',
      author: 'JSmith',
      likes: 341,
      image: '/public/images/NFT Picture3.png',
      priceEth: 4.89,
      priceUsd: 6547.84,
      onSale: true
    },
    {
      id: 2,
      name: 'Moonfall',
      author: 'JSmith',
      likes: 289,
      image: '/public/images/NFT Picture.png',
      priceEth: 3.75,
      priceUsd: 5032.15,
      onSale: true
    },
    
  ];

  let nombre = '';
  let edad = 0;
  let sexo = '';
  let viveSolo = '';
  const totalMonedas = 50;
  let opciones = [
    { key: 'pareja', label: 'Pareja', valor: 0, emoji: '❤️' },
    { key: 'familia', label: 'Familia', valor: 0, emoji: '👪' },
    { key: 'amigos', label: 'Amigos', valor: 0, emoji: '🧑‍🤝‍🧑' },
    { key: 'deporte', label: 'Deporte', valor: 0, emoji: '🏃‍♂️' },
    { key: 'estudio', label: 'Estudio', valor: 0, emoji: '📚' }
  ];

  let showArte = false;
  let arteProps = { distribucion: [], nombre: '', rareza: '', sexo: '', viveSolo: '' };
  let gallery = [];

  const colorMap = {
    pareja: '#FF220C',
    estudio: '#F3BB06',
    deporte: '#1A7102',
    familia: '#452800',
    amigos: '#3ED3BA'
  };

  // ✅ Función corregida para normalizar counts
  function normalizarCounts(counts) {
    const total = Object.values(counts).reduce((acc, val) => acc + val, 0);
    const factor = totalMonedas / total;

    let normalizado = {};
    let sum = 0;
    const keys = Object.keys(counts);

    keys.forEach(key => {
      normalizado[key] = Math.round(counts[key] * factor);
      sum += normalizado[key];
    });

    const diferencia = sum - totalMonedas;
    if (diferencia !== 0) {
      const ordenados = [...keys].sort((a, b) => normalizado[b] - normalizado[a]);
      for (let i = 0; i < Math.abs(diferencia); i++) {
        const key = ordenados[i % ordenados.length];
        normalizado[key] -= Math.sign(diferencia);
      }
    }

    return normalizado;
  }

  function generarDistribucion(counts) {
    let colores = [];
    for (const key in counts) {
      for (let i = 0; i < counts[key]; i++) {
        colores.push(colorMap[key]);
      }
    }
    return colores.sort(() => Math.random() - 0.5);
  }

  const respuestasPredeterminadas = [
    {
      nombre: "Luna",
      sexo: "femenino",
      viveSolo: "no",
      rareza: "legendario",
      counts: { pareja: 30, amigos: 25, familia: 20, estudio: 15, deporte: 10 }
    },
    {
      nombre: "Axel",
      sexo: "masculino",
      viveSolo: "sí",
      rareza: "epico",
      counts: { pareja: 10, amigos: 35, familia: 25, estudio: 20, deporte: 10 }
    },
    {
      nombre: "Sofía",
      sexo: "femenino",
      viveSolo: "no",
      rareza: "comun",
      counts: { pareja: 5, amigos: 15, familia: 40, estudio: 30, deporte: 10 }
    },
    {
      nombre: "Tomás",
      sexo: "masculino",
      viveSolo: "sí",
      rareza: "raro",
      counts: { pareja: 25, amigos: 25, familia: 25, estudio: 15, deporte: 10 }
    }
  ].map(item => {
    const normalized = normalizarCounts(item.counts);
    return {
      ...item,
      counts: normalized,
      distribucion: generarDistribucion(normalized)
    };
  });

  let filtroSexo = '';
  let filtroViveSolo = '';
  let filtroCategoria = '';

  $: galeriaCompleta = [...respuestasPredeterminadas, ...gallery];

  $: galeriaFiltrada = galeriaCompleta
    .filter(item => !filtroSexo || item.sexo === filtroSexo)
    .filter(item => !filtroViveSolo || item.viveSolo === filtroViveSolo);

  $: galeriaOrdenada = filtroCategoria
    ? [...galeriaFiltrada].sort(
        (a, b) => b.counts[filtroCategoria] - a.counts[filtroCategoria]
      )
    : galeriaFiltrada;

  $: sumDistribuidas = opciones.reduce((acc, o) => acc + o.valor, 0);
  $: monedasRestantes = totalMonedas - sumDistribuidas;

  onMount(() => {
    AOS.init({
      offset: 100,
      duration: 2000,
      easing: 'ease-in-out',
      once: false,
      mirror: true,
      anchorPlacement: 'top-bottom'
    });

    const stored = localStorage.getItem('gallery');
    gallery = stored ? JSON.parse(stored) : [];
  });

  function handleSubmit() {
    if (monedasRestantes !== 0) {
      alert(`Por favor asigna todas las monedas. Te quedan ${monedasRestantes}.`);
      return;
    }

    const counts = {};
    opciones.forEach(o => {
      counts[o.key] = o.valor;
    });

    const distribucion = generarDistribucion(counts);

    let rarezaVal = '';
    if (edad <= 18) rarezaVal = 'raro';
    else if (edad <= 40) rarezaVal = 'comun';
    else if (edad <= 80) rarezaVal = 'epico';
    else rarezaVal = 'legendario';

    arteProps = { distribucion, nombre, rareza: rarezaVal, sexo, viveSolo };
    showArte = true;

    const newItem = {
      ...arteProps,
      counts,
      timestamp: Date.now()
    };
    gallery = [newItem, ...gallery];
    localStorage.setItem('gallery', JSON.stringify(gallery));

    nombre = '';
    edad = 0;
    sexo = '';
    viveSolo = '';
    opciones.forEach(o => o.valor = 0);
  }

  function clearGallery() {
    if (confirm('¿Seguro que quieres borrar todos los NFTs guardados?')) {
      gallery = [];
      localStorage.removeItem('gallery');
    }
  }

  let frameContainer;

  async function downloadImage() {
    if (!frameContainer) return;
    const canvas = await html2canvas(frameContainer, { backgroundColor: null });
    const link = document.createElement('a');
    link.download = `${nombre || 'mi-nft'}.png`;
    link.href = canvas.toDataURL('image/png');
    link.click();
  }

  function scrollToHow() {
    const el = document.getElementById('how-container');
    if (el) {
      el.scrollIntoView({ behavior: 'smooth', block: 'center' });
    }
  }
</script>

<meta name="viewport" content="width=device-width, initial-scale=1.0">

<header class="header">
  <!-- Logo -->
  <div class="logo">
    <div class="logo-icon"></div>
    <span class="logo-text">
      <span class="logo-highlight">SOL</span>digitalArt
    </span>
  </div>

  <!-- Navegación (solo diseño) -->
  <nav class="nav">
    <a href="/explorar">Explorar</a>
    <a href="/colecciones">Colecciones</a>
    <a href="/artistas">Artistas</a>
    <a href="/crear">Crear</a>
  </nav>

  <!-- Acciones -->
  <div class="actions">
    <div class="search">
      <svg class="icon" viewBox="0 0 24 24"><path fill="currentColor" d="M10 2a8 8 0 105.293 14.293l4.207 4.207 1.414-1.414-4.207-4.207A8 8 0 0010 2zm0 2a6 6 0 110 12 6 6 0 010-12z"/></svg>
      <input type="text" placeholder="Buscar obras de arte…"/>
    </div>
    <button class="btn-user" aria-label="Perfil">
      <svg class="icon" viewBox="0 0 24 24"><path fill="currentColor" d="M12 12a5 5 0 100-10 5 5 0 000 10zm0 2c-3.866 0-7 3.134-7 7h2a5 5 0 0110 0h2c0-3.866-3.134-7-7-7z"/></svg>
    </button>
    <button class="btn-connect">Conectar</button>
  </div>
</header>

<div class="full-box">
  <div class="text-wrapper">
    <h1 class="titulo-principal">Conoce El Mundo Del</h1>
    <h1 class="titulo-principal2">Arte Digital Con SOL</h1>
    <p class="subtitulo">
      Explora el mercado de arte digital más grande del mundo. Compra, vende y descubre NFTs exclusivos de artistas talentosos de todo el mundo.
    </p>
    <div class="botones">
      <button class="boton1" on:click={scrollToHow}>
        Comenzar
        <svg class="icon icon-arrow" viewBox="0 0 24 24">
          <path fill="none" stroke="currentColor" stroke-width="2" d="M5 12h14M13 5l7 7-7 7"/>
        </svg>
      </button>
      <button class="boton2">
        <svg class="icon icon-play" viewBox="0 0 24 24">
          <path fill="currentColor" d="M8 5v14l11-7z"/>
        </svg>
        Ver Tutorial
      </button>
    </div>

    <!-- Nueva sección de estadísticas -->
    <div class="stats">
      <div class="stat-item">
        <span class="stat-number">47.3K+</span>
        <span class="stat-label">Obras de Arte</span>
      </div>
      <div class="stat-item">
        <span class="stat-number">15.2K+</span>
        <span class="stat-label">Artistas</span>
      </div>
      <div class="stat-item">
        <span class="stat-number">89.1K+</span>
        <span class="stat-label">Coleccionistas</span>
      </div>
    </div>
  </div>

  <div class="image-wrapper">
    <img src="/public/images/NFT2.png" alt="Living Of The Art" />
    <img src="/public/images/NFT1.png" alt="Living Of The Art" />
    <img src="/public/images/NFT3.png" alt="Living Of The Art" />
  </div>
</div>

<section id="how-container" class="how-container">
  <h2 class="how-title" data-aos="fade-up" data-aos-duration="1000">
    ¿Cómo <span class="gradient-text">Funciona</span>?
  </h2>
  <p class="how-subtitle" data-aos="fade-up" data-aos-duration="1000">
    Sigue estos simples pasos para comenzar tu viaje en el mundo de los NFTs
  </p>
  <div class="how-steps">
    <div class="how-step"
         data-aos="fade-right"
         data-aos-delay="100">
      <div class="how-step-number">01</div>
      <div class="how-step-title">Completa la encuesta</div>
      <div class="how-step-desc">
        Completa la encuesta para crear tu arte digital y hacerlo único.
      </div>
    </div>

    <div class="how-step"
         data-aos="fade-right"
         data-aos-delay="400">
      <div class="how-step-number">02</div>
      <div class="how-step-title">Descarga tu Arte Digital</div>
      <div class="how-step-desc">
        Completa la encuesta para personalizar tu arte digital y hacerlo único
      </div>
    </div>

    <div class="how-step"
         data-aos="fade-right"
         data-aos-delay="700">
      <div class="how-step-number">03</div>
      <div class="how-step-title">Descarga y comparte</div>
      <div class="how-step-desc">
        Descarga tu Arte figital personalizado y compártelo con tus amigos
      </div>
    </div>

    <div class="how-step"
         data-aos="fade-right"
         data-aos-delay="1000">
      <div class="how-step-number">04</div>
      <div class="how-step-title">Colecciona y vende</div>
      <div class="how-step-desc">
        Construye tu colección y revende tus NFTs cuando quieras
      </div>
    </div>
  </div>
</section>


<h2 class="how-title" data-aos="fade-up">
   Completa La encuesta para crear <span class="gradient-text"> Tu Arte Digital</span> 
</h2>
<p class="how-subtitle" data-aos="fade-up" data-aos-delay="100">
  Responde algunas preguntas para personalizar tu NFT y hacerlo único.
</p>
<section class="encuesta-contenedor" data-aos="fade-up">
  <!-- Título encima del formulario -->
  <!-- 2) Card/Formulario -->
  <form 
    class="formulario-personalizacion" 
    on:submit|preventDefault={handleSubmit}
  >
    <!-- Header de la card -->
    <div class="card-header">
      Formulario de Personalización, ¡comencemos!
    </div>

    <!-- 3) Nombre / Edad en dos columnas -->
    <div class="grid-dos-inputs">
      <div class="campo">
        <label for="nombre">Nombre</label>
        <input
          id="nombre"
          type="text"
          placeholder="Tu nombre"
          bind:value={nombre}
          required
        />
      </div>
      <div class="campo">
        <label for="edad">Edad</label>
        <input
          id="edad"
          type="number"
          min="0"
          placeholder="Tu edad"
          bind:value={edad}
          required
        />
      </div>
    </div>

    <!-- 4) Selects simples -->
    <div class="campo">
      <label for="viveSolo">¿Vives solo?</label>
      <select id="viveSolo" bind:value={viveSolo} required>
        <option value="" disabled selected>Selecciona una opción</option>
        <option value="sí">Sí</option>
        <option value="no">No</option>
      </select>
    </div>
    <div class="campo">
      <label class= "sexo" for="sexo">Sexo</label>
      <select id="sexo" bind:value={sexo} required>
        <option value="" disabled selected>Selecciona tu sexo</option>
        <option value="masculino">Masculino</option>
        <option value="femenino">Femenino</option>
        <option value="otro">Otro</option>
      </select>
    </div>

    <!-- 5) Distribución de monedas con sliders -->
    <fieldset class="campo-distribucion">
      <legend><span class="icon">🪙</span> Reparte tus 50 monedas según tus planes de fin de semana</legend>
      {#each opciones as opcion}
        <div class="subcampo">
          <label for={opcion.key}>
            {opcion.label}
          </label>
          <input
            id={opcion.key}
            type="range"
            min="0"
            max={totalMonedas}
            bind:value={opcion.valor}
          />
          <span class="valor">{opcion.valor}</span>
        </div>
      {/each}

      <div class="restantes">
        Monedas restantes:
        <span class:alerta={monedasRestantes < 0}>
          {monedasRestantes}
        </span>
      </div>
    </fieldset>

    <!-- 6) Botón Generar -->
    <button type="submit" disabled={monedasRestantes !== 0}>
      ✨ Generar Mi NFT Personalizado
    </button>
  </form>
</section>

{#if showArte}
    <div class="arte-principal">
      <!-- Header con título -->
      
      <!-- Contenedor con borde y badge -->
      <div class="frame-wrapper">
        <div class="nft-frame-container" bind:this={frameContainer}>
          <ArteDigital
            distribucion={arteProps.distribucion}
            nombre={arteProps.nombre}
            rareza={arteProps.rareza}
            sexo={arteProps.sexo}
            viveSolo={arteProps.viveSolo}
          />
        </div>
        <span class="badge">Único</span>
      </div>
      <!-- Botones de acción -->
      <div class="actions">
        <button type="button" class="btn download" on:click={downloadImage}>
          <svg class="icon" viewBox="0 0 24 24"><path fill="currentColor" d="M5 20h14v-2H5v2zm7-18L5.33 9h3.67v4h4V9h3.67L12 2z"/></svg>
          Descargar
        </button>
        <button type="button" class="btn share">
          <svg class="icon" viewBox="0 0 24 24"><path fill="currentColor" d="M18 16.08c-.76 0-1.44.3-1.96.77L8.91 12.7c.05-.23.09-.46.09-.7s-.04-.47-.09-.7l7.02-4.11c.54.5 1.25.82 2.07.82 1.66 0 3-1.34 3-3s-1.34-3-3-3-3 1.34-3 3c0 .24.04.47.09.7L8.09 9.81C7.55 9.3 6.84 9 6 9c-1.66 0-3 1.34-3 3s1.34 3 3 3c.84 0 1.55-.3 2.09-.81l7.14 4.18c-.05.21-.08.43-.08.66 0 1.66 1.34 3 3 3s3-1.34 3-3-1.34-3-3-3z"/></svg>
          Compartir
        </button>
      </div>
    </div>
{/if}


<h2 class="how-title" data-aos="fade-up">
   Referencias de <span class="gradient-text"> Tu Arte Digital</span> 
</h2>
<p class="how-subtitle" data-aos="fade-up" data-aos-delay="100">
  Aquí están las referencias visuales que representan tu arte digital personalizado. Cada elemento refleja tus respuestas y preferencias.
</p>
<section class="referencias-minimal" data-aos="fade-up" data-aos-delay="100">
  <div class="referencias-layout">

    <!-- 🟣 Género -->
    <div class="referencia-bloque generoH">
      <img src="/images/top hombre.png" alt="Hombre" />
      <span>Hombre</span>
    </div>

    <div class="referencia-bloque generoM">
      <img src="/images/top mujer.png" alt="Mujer" />
      <span>Mujer</span>
    </div>
    <h4 class="titulo-distribucion">Cada uno de los cuadrados de tu arte representa una moneda gastada</h4>
    <!-- 🏠 Vivienda -->
    <div class="referencia-bloque vivienda">
      <img class="tornillo1" src="/images/tornillo.png" alt="Vive solo" />
      <span>Vive solo</span>
    </div>
    <!-- 🟨 Distribución (cuadrados) -->
    <div class="referencia-bloque distribucion pareja">
      <div class="square pareja"></div>
      <span>Pareja</span>
    </div>

    <div class="referencia-bloque distribucion estudio">
      <div class="square estudio"></div>
      <span>Estudio</span>
    </div>

    <div class="referencia-bloque distribucion deporte">
      <div class="square deporte"></div>
      <span>Deporte</span>
    </div>

    <div class="referencia-bloque distribucion familia">
      <div class="square familia"></div>
      <span>Familia</span>
    </div>

    <div class="referencia-bloque distribucion amigos">
      <div class="square amigos"></div>
      <span>Amigos</span>
    </div>

    <!-- 🔶 Rareza -->
    <div class="referencia-bloque rareza raro">
      <div class="carta raro">
        <strong>Raro</strong>
        <span>0–18 años</span>
      </div>
    </div>

    <div class="referencia-bloque rareza comun">
      <div class="carta comun">        <strong>Común</strong>
        <span>18–40 años</span>
      </div>
    </div>

    <div class="referencia-bloque rareza epico">
      <div class="carta epico">
        <strong>Épico</strong>
        <span>40–80 años</span>
      </div>
    </div>

    <div class="referencia-bloque rareza legendario">
      <div class="carta legendario">
        <strong>Legendario</strong>
        <span>80+ años</span>
      </div>
    </div>

  </div>
</section>

<!-- Galería -->
<section class="gallery">
  <h2 class="how-title">
    <span class="gradient-text">Galería de Arte Digital</span>
  </h2>
  <p class="how-subtitle">
    Explora las obras de arte digitales creadas por nuestra comunidad.
  </p>
  <div class="gallery-filters">
    <select bind:value={filtroSexo}>
      <option value="">— Sexo —</option>
      <option value="masculino">Hombre</option>
      <option value="femenino">Mujer</option>
      <option value="otro">Otro</option>
    </select>
    <select bind:value={filtroViveSolo}>
      <option value="">— Vive Solo —</option>
      <option value="sí">Sí</option>
      <option value="no">No</option>
    </select>
    <select bind:value={filtroCategoria}>
      <option value="">— Más Gastado en —</option>
      {#each Object.keys(colorMap) as key}
        <option value={key}>
          {key.charAt(0).toUpperCase() + key.slice(1)}
        </option>
      {/each}
    </select>
  </div>
  <div class="gallery-grid">
  {#each galeriaOrdenada as item (item.nombre + item.rareza)}
    <div class="gallery-item">

      <div class="arte-wrapper">
        <ArteDigital
          distribucion={item.distribucion}
          sexo={item.sexo}
          viveSolo={item.viveSolo}
        />
      </div>
      <div class="arte-texto">
        <h3 class="arte-nombre">{item.nombre}</h3>
        <p class="arte-rareza">{item.rareza}</p>
      </div>


    </div>
  {/each}
</div>

</section>

<section class="market-section">
  <p class="subheading" data-aos="fade-up">Artes mas reconocidos del minteo</p>
  <div class="market-header" data-aos="fade-up">
    <h2>Mercado de Arte Digital</h2>
    <button class="view-all-btn">View All</button>
  </div>

  <div class="cards-grid">
    <!-- Card 1 -->
    <div class="card" data-aos="zoom-in">
      <div class="card-image">
        <img src="/public/images/referencia1.png" alt="Tetriz" />
        <div class="likes">
          <svg viewBox="0 0 24 24">
            <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 
                     3c1.74 0 3.41 0.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 
                     19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 
                     11.54L12 21.35z" />
          </svg>
          <span>341</span>
        </div>
      </div>
      <div class="card-body">
        <div class="card-title-author">
          <img class="author-avatar" src="/public/images/avatar1.png" alt="JSmith" />
          <div>
            <h3>Pixel Read</h3>
            <div class="author">@JSmith</div>
          </div>
        </div>
        <div class="status-price">
          <div class="status">
            On Sale
            <svg viewBox="0 0 24 24">
              <path d="M13.5 2C9.36 2 6 5.36 6 9.5c0 5.25 7.5 12.5
                       7.5 12.5s7.5-7.25 7.5-12.5C21 5.36 17.64
                       2 13.5 2z" />
            </svg>
          </div>
          <div class="price">
            <span class="eth">4.89 ETH</span>
            <span class="usd">($6,547.84)</span>
          </div>
        </div>
        <button class="buy-btn">Buy Now</button>
      </div>
    </div>

    <!-- Card 2 -->
    <div class="card" data-aos="zoom-in">
      <div class="card-image">
        <img src="/images/referencia2.png" alt="Moonfall" />
        <div class="likes">
          <svg viewBox="0 0 24 24">
            <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 
                     3c1.74 0 3.41 0.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 
                     19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 
                     11.54L12 21.35z" />
          </svg>
          <span>289</span>
        </div>
      </div>
      <div class="card-body">
        <div class="card-title-author">
          <img class="author-avatar" src="/images/avatar1.png" alt="JSmith" />
          <div>
            <h3>Pixel Banana</h3>
            <div class="author">@JSmith</div>
          </div>
        </div>
        <div class="status-price">
          <div class="status">
            On Sale
            <svg viewBox="0 0 24 24">
              <path d="M13.5 2C9.36 2 6 5.36 6 9.5c0 5.25 7.5 12.5
                       7.5 12.5s7.5-7.25 7.5-12.5C21 5.36 17.64
                       2 13.5 2z" />
            </svg>
          </div>
          <div class="price">
            <span class="eth">3.75 ETH</span>
            <span class="usd">($5,032.15)</span>
          </div>
        </div>
        <button class="buy-btn">Buy Now</button>
      </div>
    </div>

    <!-- Card 3 -->
    <div class="card" data-aos="zoom-in">
      <div class="card-image">
        <img src="/images/referencia3.png" alt="Wired Human" />
        <div class="likes">
          <svg viewBox="0 0 24 24">
            <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 
                     3c1.74 0 3.41 0.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 
                     19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 
                     11.54L12 21.35z" />
          </svg>
          <span>415</span>
        </div>
      </div>
      <div class="card-body">
        <div class="card-title-author">
          <img class="author-avatar" src="/images/avatar1.png" alt="JSmith" />
          <div>
            <h3>Pixel chart</h3>
            <div class="author">@JSmith</div>
          </div>
        </div>
        <div class="status-price">
          <div class="status">
            On Sale
            <svg viewBox="0 0 24 24">
              <path d="M13.5 2C9.36 2 6 5.36 6 9.5c0 5.25 7.5 12.5
                       7.5 12.5s7.5-7.25 7.5-12.5C21 5.36 17.64
                       2 13.5 2z" />
            </svg>
          </div>
          <div class="price">
            <span class="eth">4.20 ETH</span>
            <span class="usd">($5,462.10)</span>
          </div>
        </div>
        <button class="buy-btn">Buy Now</button>
      </div>
    </div>

    <!-- Card 4 -->
    <div class="card" data-aos="zoom-in">
      <div class="card-image">
        <img src="/images/referencia4.png" alt="Racer-To-Go" />
        <div class="likes">
          <svg viewBox="0 0 24 24">
            <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 
                     3c1.74 0 3.41 0.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 
                     19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 
                     11.54L12 21.35z" />
          </svg>
          <span>312</span>
        </div>
      </div>
      <div class="card-body">
        <div class="card-title-author">
          <img class="author-avatar" src="/images/avatar1.png" alt="JSmith" />
          <div>
            <h3>Pixel F1</h3>
            <div class="author">@JSmith</div>
          </div>
        </div>
        <div class="status-price">
          <div class="status">
            On Sale
            <svg viewBox="0 0 24 24">
              <path d="M13.5 2C9.36 2 6 5.36 6 9.5c0 5.25 7.5 12.5
                       7.5 12.5s7.5-7.25 7.5-12.5C21 5.36 17.64
                       2 13.5 2z" />
            </svg>
          </div>
          <div class="price">
            <span class="eth">4.89 ETH</span>
            <span class="usd">($6,547.84)</span>
          </div>
        </div>
        <button class="buy-btn">Buy Now</button>
      </div>
    </div>

    <!-- Duplica más cards aquí si necesitas -->
  </div>
</section>

<section class="digital-future" >
  <h2 class="future-title" data-aos="fade-up">El Futuro es <span class="gradient-text">Digital</span></h2>
  <p class="future-subtitle" data-aos="fade-up">
    Los NFTs han transformado la forma en que percibimos la propiedad digital,<br />
    creando nuevas economías y comunidades en el metaverso.
  </p>

  <div class="future-cards">
    <div class="future-card economic" data-aos="fade-right" data-aos-delay="100">
      <div class="future-icon">$</div>
      <h3>Valor Económico</h3>
      <p>Mercado de miles de millones<br />creando nuevas oportunidades</p>
    </div>

    <div class="future-card community" data-aos="fade-right" data-aos-delay="200">
      <div class="future-icon">👥</div>
      <h3>Comunidad</h3>
      <p>Millones de usuarios<br />construyendo el futuro digital</p>
    </div>

    <div class="future-card innovation" data-aos="fade-right" data-aos-delay="300">
      <div class="future-icon">⚡</div>
      <h3>Innovación</h3>
      <p>Tecnología que redefine la<br />creatividad y propiedad</p>
    </div>
  </div>
</section>
<footer class="footer">
  <div class="footer-container">

    <!-- Logo + descripción + sociales -->
    <div class="footer-logo-col">
      <div class="footer-logo">
        <div class="logo-icon"></div>
        <span>SOLdigitalArt</span>
      </div>
      <p class="footer-desc">
        Explora, crea y colecciona arte digital único. Conecta con artistas de todo el mundo y descubre obras exclusivas en nuestra plataforma.
      </p>
      <div class="footer-social">
        <a href="https://www.youtube.com/@Rarible" aria-label="YouTube">
          <img src="public/images/yt.png" alt="YouTube" />
        </a>
        <a href="https://x.com/rarible" aria-label="Twitter">
          <img src="public/images/tw.png" alt="Twitter" />
        </a>
        <a href="https://www.facebook.com/rarible/?locale=es_LA" aria-label="Facebook">
          <img src="public/images/fb.png" alt="Facebook" />
        </a>
        <a href="https://rarible.com/" aria-label="Google+">
          <img src="public/images/gg.png" alt="Google+" />
        </a>
      </div>
    </div>

    <!-- Columna About -->
    <div class="footer-links">
      <h4>About</h4>
      <ul>
        <li><a href="https://rarible.com/blog/">About NFT</a></li>
        <li><a href="https://rarible.com/community-guidelines">Live Auctions</a></li>
        <li><a href="https://rarible.com/explore/all/collections">NFT Blog</a></li>
        <li><a href="https://rarible.com/explore/all/collections">Activity</a></li>
      </ul>
    </div>

    <!-- Columna Support -->
    <div class="footer-links">
      <h4>Support</h4>
      <ul>
        <li><a href="https://static.rarible.com/privacy.pdf">Help & Support</a></li>
        <li><a href="https://static.rarible.com/terms.pdf">Item Details</a></li>
        <li><a href="https://rarible.com/branding">Author Profile</a></li>
        <li><a href="https://opensea.io/collection/deeple-stuff">Collection</a></li>
      </ul>
    </div>

    <!-- Copyright full width -->
    <div class="footer-copy">
      All rights reserved @matteoosuna and @manuelpignataro
    </div>
  </div>
</footer>