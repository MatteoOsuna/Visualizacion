<script>
  // Datos del formulario
  let nombre = '';
  let edad = '';
  let sexo = '';
  let viveSolo = '';
  let distribucion = {
    deporte: 10,
    amigos: 10,
    familia: 10,
    estudio: 10,
    pareja: 10
  };

  // Reacciones automáticas
  $: totalMonedas = Object.values(distribucion).reduce((sum, val) => sum + val, 0);
  $: monedasRestantes = 50 - totalMonedas;

  // Ajusta una categoría sin pasarse de 50 monedas
  function handleDistribucionChange(categoria, nuevoValor) {
    const otherTotal = totalMonedas - distribucion[categoria];
    if (otherTotal + nuevoValor <= 50) {
      distribucion = { ...distribucion, [categoria]: nuevoValor };
    }
  }

  // Envío del formulario
  function handleSubmit() {
    if (!nombre || !edad || !sexo || !viveSolo) {
      alert('Por favor completa todos los campos');
      return;
    }
    if (totalMonedas !== 50) {
      alert('Debes distribuir exactamente 50 monedas');
      return;
    }
    alert(`Tu NFT personalizado "${nombre}" ha sido creado exitosamente`);
    console.log('Datos del NFT:', { nombre, edad, sexo, viveSolo, distribucion });
  }
</script>

<section class="py-20 bg-gradient-to-b from-background to-muted/20">
  <div class="container max-w-4xl mx-auto">
    <div class="text-center mb-12">
      <h2 class="text-3xl lg:text-4xl font-bold mb-4">
        Genera tu
        <span class="bg-gradient-to-r from-purple-600 to-pink-600 bg-clip-text text-transparent">
          NFT Personalizado
        </span>
      </h2>
      <p class="text-lg text-muted-foreground">
        Responde estas preguntas y crearemos un NFT único basado en tu personalidad
      </p>
    </div>

    <div class="bg-card/50 backdrop-blur border border-border/50 p-8 rounded-xl">
      <div class="text-center mb-6">
        <h3 class="text-xl font-semibold flex items-center justify-center gap-2">
          <!-- Aquí podrías usar tu icono SVG -->
          ✨ Formulario de Personalización
        </h3>
      </div>

      <form on:submit|preventDefault={handleSubmit} class="space-y-8">
        <!-- Información básica -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div class="space-y-2">
            <label for="nombre" class="block text-sm font-medium">Nombre</label>
            <input
              id="nombre"
              type="text"
              placeholder="Tu nombre"
              bind:value={nombre}
              class="w-full rounded-md border px-3 py-2"
            />
          </div>
          <div class="space-y-2">
            <label for="edad" class="block text-sm font-medium">Edad</label>
            <input
              id="edad"
              type="number"
              placeholder="Tu edad"
              bind:value={edad}
              class="w-full rounded-md border px-3 py-2"
            />
          </div>
        </div>

        <!-- Sexo -->
        <div class="space-y-2">
          <label for="sexo" class="block text-sm font-medium">Sexo</label>
          <select
            id="sexo"
            bind:value={sexo}
            class="w-full rounded-md border px-3 py-2"
          >
            <option value="" disabled>Selecciona tu sexo</option>
            <option value="masculino">Masculino</option>
            <option value="femenino">Femenino</option>
            <option value="otro">Otro</option>
          </select>
        </div>

        <!-- Vive solo -->
        <div class="space-y-2">
          <label for="viveSolo" class="block text-sm font-medium">¿Vives solo?</label>
          <select
            id="viveSolo"
            bind:value={viveSolo}
            class="w-full rounded-md border px-3 py-2"
          >
            <option value="" disabled>Selecciona una opción</option>
            <option value="si">Sí</option>
            <option value="no">No</option>
          </select>
        </div>

        <!-- Distribución de monedas -->
        <div class="space-y-6">
          <div class="text-center">
            <span class="text-lg font-semibold flex items-center justify-center gap-2">
              <!-- Icono de monedas -->
              💰 Distribuye 50 Monedas
            </span>
            <p class="text-sm text-muted-foreground mt-1">
              Monedas restantes:
              <span class={`font-bold ${monedasRestantes === 0 ? 'text-green-500' : 'text-orange-500'}`}>
                {monedasRestantes}
              </span>
            </p>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            {#each Object.entries(distribucion) as [categoria, valor], i (categoria)}
              <div class="space-y-2">
                <div class="flex justify-between">
                  <label class="capitalize text-sm font-medium" for={`slider-${categoria}`}>{categoria}</label>
                <input
                  id={`slider-${categoria}`}
                  type="range"
                  min="0"
                  max="50"
                  step="1"
                  value={valor}
                  on:input={(e) => handleDistribucionChange(categoria, +e.currentTarget.value)}
                  class="w-full"
                />
                  class="w-full"
                />
              </div>
            {/each}
          </div>
        </div>

        <button
          type="submit"
          class="w-full rounded-lg py-3 text-white font-semibold
                 bg-gradient-to-r from-purple-600 to-pink-600
                 hover:from-purple-700 hover:to-pink-700
                 disabled:opacity-50"
          disabled={totalMonedas !== 50}
        >
          ✨ Generar Mi NFT Personalizado
        </button>
      </form>
    </div>
  </div>
</section>
