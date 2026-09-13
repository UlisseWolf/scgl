# Modifiche apportate

Tutte le modifiche sono state validate compilando il progetto con un cross-compiler
i686-w64-mingw32 (g++ 13, MinGW-w64) — la build completa (`cmake --build .`) risulta
pulita, senza errori. Questo garantisce che il codice sia sintatticamente e
semanticamente corretto per il compilatore, **non** che sia stato verificato in gioco:
qui non è disponibile Windows, DirectX, una GPU reale né SimCity 4, quindi nessuna di
queste modifiche è stata testata a runtime. Vanno validate su una macchina reale prima
di essere pubblicate/merged.

## Issue #7 — Alpha blending delle auto rotto (RISOLTA)

File: `scgl/GLStateManager.cpp`, `GLStateManager::AlphaMultiplier`.

Bug reale trovato: l'alpha veniva inviato alla GPU (`glMaterialfv`) solo quando il
color-tracking (`GL_COLOR_MATERIAL`, attivato da `EnableVertexColors`) era attivo —
ma è esattamente la situazione in cui quel valore viene sovrascritto per-vertice e
quindi ignorato. Gli oggetti senza vertex color array (come le auto, illuminate ma
senza colori per-vertice) non ricevevano mai l'alpha aggiornato: risultato, comparivano
e scomparivano di scatto invece di dissolversi gradualmente. Rimossa la condizione
errata: l'alpha viene ora sempre inviato, dato che `GL_LIGHTING` resta permanentemente
attivo (vedi `cGDriver::Init`) e quindi il materiale incide sempre sul colore finale.

## Issue #9 — Draw call batching (implementata, da validare in gioco)

File principali: `scgl/GLStateManager.h/.cpp`, più punti di flush sparsi in
`cGDriver.cpp`, `cGDriver_Textures.cpp`, `cGDriver_Viewport.cpp`,
`ext/cGDriver_BufferRegions.cpp`, `ext/cGDriver_Lighting.cpp`, `ext/cGDriver_Snapshot.cpp`.

Le chiamate `DrawElements` consecutive con stesso tipo di primitiva/indice vengono
accumulate e inviate insieme con `glMultiDrawElements` (caricata dinamicamente,
con fallback al vecchio comportamento se l'estensione non è disponibile), invece di
una `glDrawElements` per ogni chiamata. Perché il batching non alteri l'aspetto visivo,
ogni punto del codice che cambia stato GL (blend, texture, matrici, luci, letture del
framebuffer, ecc.) forza prima l'invio ("flush") del batch in sospeso: questo è il
pezzo più delicato e più a rischio di questa modifica — se un punto di mutazione dello
stato fosse stato dimenticato, il sintomo sarebbe un rendering silenziosamente
sbagliato (colori/texture applicati al gruppo di draw call sbagliato), non un crash.
Ho passato in rassegna sistematicamente i file del driver per individuare questi punti,
ma **raccomando fortemente un test visivo in gioco** prima di considerarla definitiva.

## Issue #10 — GDriverVertexBufferExtension (implementata, contratto da verificare)

File: `scgl/ext/cGDriver_VertexBuffers.cpp`, `scgl/GLSupport.h/.cpp`, `scgl/cGDriver.h`.

L'interfaccia `cIGZGDriverVertexBufferExtension` non ha alcuna documentazione nè nel
codice nè altrove nel repository, e l'unico riferimento è il PrimitiveManager (closed
source) del driver DirectX del gioco. Ho implementato un pool di Vertex/Index Buffer
Object OpenGL reali con un'interpretazione plausibile ma **non verificata** del
contratto di chiamata (spiegata nel commento in testa al file). Non essendo disponibile
il gioco per tracciare le chiamate reali, questa resta un'ipotesi di lavoro solida ma
da validare — nel peggiore dei casi un'ipotesi sbagliata qui produce geometria non
disegnata, non un crash, il che l'ho considerato un compromesso di sicurezza accettabile.

## Issue #12 — Region view pixellata (NON risolta, solo investigata)

Vedi `NOTES_ISSUE_12.md`. È un bug report senza causa nota, legato a una combinazione
hardware/driver (Radeon + Mesa + Linux/Proton) impossibile da riprodurre in questo
ambiente. Ho scartato un paio di ipotesi plausibili dopo aver letto il codice, ma non
ho trovato una causa da poter correggere con sicurezza, e non ho voluto inventare un
fix che non potevo verificare.
