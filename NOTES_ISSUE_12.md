# Issue #12 — "The region view turns into a pixellated mess"

## Stato: non risolta — solo investigazione

A differenza delle issue #7, #9 e #10, questa non è stata risolta con una modifica
di codice. Si tratta di una segnalazione senza causa nota nemmeno da parte di chi
l'ha aperta ("Not sure what exactly caused it"), su una combinazione hardware/driver
specifica (Radeon RX 5700 XT, Mesa RadeonSI, Linux via Proton) che non è possibile
riprodurre in questo ambiente: qui non c'è una GPU, non c'è SimCity 4, e non c'è modo
di osservare l'output renderizzato.

Applicare una "correzione" senza poterla verificare visivamente sarebbe irresponsabile:
rischierebbe di introdurre un cambiamento che sembra plausibile sulla carta ma non
risolve nulla, o peggio nasconde il problema reale.

## Piste investigate e scartate

- **`GL_NV_texture_env_combine4`**: è un'estensione proprietaria NVIDIA usata per il
  combine a 4 stadi delle texture (probabile per il blending del terreno). SCGL la
  rileva correttamente (`cGDriver_Init.cpp`) e comunica al gioco tramite
  `supportsNvTextureEnvCombine4` se è disponibile. Su Mesa/AMD questa estensione non
  è supportata, ma poiché il gioco **riceve onestamente `false`** per questa capacità,
  dovrebbe evitare quel percorso di rendering — non sembra la causa diretta, a meno
  che il gioco stesso gestisca male l'assenza della capacità (cosa che non possiamo
  verificare da qui, essendo codice closed-source del gioco).
- **Buffer region / framebuffer per la region view**: il codice in
  `ext/cGDriver_BufferRegions.cpp` sceglie il formato del renderbuffer in base alla
  profondità colore della modalità video corrente (`GL_RGBA8`/`GL_RGB5_A1` e
  `GL_DEPTH_COMPONENT24`/`16`). Non ho trovato un'incongruenza evidente, ma è un'area
  che vale la pena testare con un debugger grafico (vedi sotto).
- **Formato pixel/contesto GL**: identico tra region view e city view (stessa finestra,
  stesso contesto), quindi non spiega perché solo la region view sia rotta.

## Cosa servirebbe per una diagnosi vera

1. Una **cattura RenderDoc o apitrace** dell'avvio del gioco su un sistema Linux/Mesa/AMD,
   per vedere esattamente quali chiamate GL producono l'immagine "pixellata" e con quali
   texture/stati.
2. Le stringhe `GL_VENDOR` / `GL_RENDERER` / `GL_VERSION` e la lista di estensioni
   riportate da Mesa su quella scheda (si possono ottenere con `glxinfo` o aggiungendo
   un log temporaneo in `cGDriver::Init`).
3. Conferma se il problema si presenta anche **senza** le mod elencate dal segnalatore
   (NAM, SC4Fix, SC4GraphicsOptions, ecc.), per escludere interazioni con quelle.

Consiglio di chiedere queste informazioni a chi ha aperto la issue prima di investire
altro tempo in ipotesi non verificabili.
