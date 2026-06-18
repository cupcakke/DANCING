Az alábbiakban a teljes wiki magyar fordítása következik, minden aloldallal együtt.

---

1. PROJEKT ÁTTEKINTÉS

A DANCING projekt egy hibrid kvantum-relációs számítási keretrendszer, amelyet összetett tudásstruktúrák reprezentálására, optimalizálására és ellenőrzésére terveztek. A rendszer egy alapvető relációs gráfmodellt kvantumállapot-szimulációval, temporális verziókezeléssel és formális ellenőrzéssel integrálva lehetővé teszi a nagy pontosságú következtetést többdimenziós adatkészleteken.

A rendszer a Zig programozási nyelven épül, hangsúlyt fektetve a memóriabiztonságra és a nagy teljesítményű végrehajtásra speciális feldolgozó egységeken (RGPU, VPU) és egy egyedi feladatütemező kernelen (ChaosCore) keresztül.

A rendszerintegrációs térkép bemutatja, hogyan lépnek kölcsönhatásba a főbb szoftverkomponensek, a ZRuntime koordinátor és a SelfSimilarRelationalGraph köré szervezve.

Főbb alrendszerek:

1. Alapgráf Réteg (nsir_core)
A projekt alapja a SelfSimilarRelationalGraph. A szabványos tulajdonsággráfoktól eltérően a csomópontok kvantumállapotokat (Qubit) tartalmaznak, az élek pedig EdgeQuality metrikákkal rendelkeznek (pl. összefonódott, koherens, fraktál). Ez a réteg kezeli az alapvető topológiát és a kvantumkapu-alkalmazásokat.
Kulcsentitások: Node, Edge, Qubit, SelfSimilarRelationalGraph.

2. Optimalizálás és Következtetés
A rendszer az Összefonódott Sztochasztikus Szimmetria Optimalizálót (ESSO) alkalmazza a gráfenergia minimalizálásához. Ezt egy ReasoningOrchestrator felügyeli, amely hierarchikus következtetési szinteket kezel, a helyi perturbációktól a meta-fázis felügyeletig.
Kulcsentitások: EntangledStochasticSymmetryOptimizer, ReasoningOrchestrator, FractalTree.

3. Végrehajtási Infrastruktúra
A nagy teljesítményű végrehajtást a ChaosCoreKernel biztosítja, amely kezeli a feladatütemezést és a Tartalom-Címezhető Tárolást (CAS). A párhuzamos gráfműveletek a RelationalGraphProcessingUnit (RGPU) egységre kerülnek, míg a SIMD numerikus munkát a VectorProcessingUnit (VPU) kezeli.
Kulcsentitások: ChaosCoreKernel, RelationalGraphProcessingUnit, SimdVector.

4. Ellenőrzés és Biztonság
A verem tartalmaz egy FormalVerificationEngine-t a gráfinvariánsokhoz és egy TypeTheoryEngine-t a lineáris erőforrás-ellenőrzéshez. Az adatvédelmet a ZKInferenceProver és az adatkészlet-elhomályosítási technikák biztosítják.
Kulcsentitások: FormalVerificationEngine, ZKInferenceProver, TypeTheoryEngine.

Adatfolyam: A bemenettől az ellenőrzött kimenetig

| Szakasz | Elsődleges kódentitás | Fájlhivatkozás |
| :--- | :--- | :--- |
| Betöltés | CREVPipeline | crev_pipeline.zig:98-109 |
| Koordináció | ZRuntime | z_runtime.zig:49-58 |
| Tárolás | SelfSimilarRelationalGraph | nsir_core.zig:250-265 |
| Optimalizálás | EntangledStochasticSymmetryOptimizer | esso_optimizer.zig:86-97 |
| Hardver | RelationalGraphProcessingUnit | r_gpu.zig:73-83 |
| Ellenőrzés | FormalVerificationEngine | formal_verification.zig:14-20 |

Gyermekszakaszok:
- Architektúra Áttekintés: Az öt réteg részletes leírása (Alap, Végrehajtás, Optimalizálás, Ellenőrzés, Hardver).
- Első Lépések és Modultérkép: Fordítási utasítások, C API belépési pontok és a Zig modulfa átfogó könyvtára.

---

Tartalom táblázata:
1: Projekt Áttekintés
1.1: Architektúra Áttekintés
1.2: Első Lépések és Modultérkép
2: Alapgráf Motor (nsir_core)
2.1: SelfSimilarRelationalGraph: Csomópontok, Élek és Kvantumállapotok
2.2: C API (jaide_ Interfész)
2.3: Temporális Gráf és Verzionált Állapot
3: Optimalizálási Motorok
3.1: Összefonódott Sztochasztikus Szimmetria Optimalizáló (ESSO)
3.2: Érvelési Orchestrátor
3.3: Fraktál Csomópont Adatrendszer (FNDS)
4: Végrehajtási és Feldolgozási Infrastruktúra
4.1: ChaosCoreKernel és Tartalom-Címezhető Tárolás
4.2: Relációs Gráf Feldolgozó Egység (RGPU)
4.3: Vektor Feldolgozó Egység (VPU) és Numerikus Motor
4.4: Jelterjedési Motor
5: Tudáskinyerés és Memória
5.1: CREV Csővezeték: Tudáskinyerés és Indexelés
5.2: ZRuntime: Változó és Relációs Logikai Környezet
5.3: Meglepetés Memóriakezelő
6: Kvantumszámítási Réteg
6.1: Kvantumlogika Szimulációs Motor
6.2: Kvantumhardver Absztrakció és IBM Integráció
6.3: Kvantumfeladat Adapter
7: Ellenőrzési és Biztonsági Verem
7.1: Formális Ellenőrzési Motor
7.2: Típuselmélet és Lineáris Erőforrás Ellenőrzés
7.3: Biztonsági Bizonyítékok és Információáramlás Elemzés
7.4: Nulla-Tudás Ellenőrzés és Ellenőrzött Következtetés
7.5: Adatkészlet Elhomályosítás és Adatvédelmi Keretrendszer
8: Biztonsági Primitívek és Keresztmetsző Segédprogramok
8.1: Biztonsági Primitívek (safety.zig)
8.2: Modulregiszter és Rendszerintegráció (mod.zig)
9: Szójegyzék

---

1.1. ARCHITEKTÚRA ÁTTEKINTÉS

A DANCING keretrendszer egy rétegzett architektúrát valósít meg, amelyet hibrid kvantum-relációs számításhoz terveztek. Áthidalja a magas szintű szemantikai következtetést az alacsony szintű kvantumvégrehajtással és a formális ellenőrzéssel. A rendszer úgy van felépítve, hogy az információt egy csővezetéken dolgozza fel, amely relációs tudáskinyeréssel kezdődik és ellenőrzött, kvantum-optimalizált gráfállapotokkal végződik.

Rendszerhierarchia

Az architektúra öt elsődleges funkcionális rétegre oszlik, amelyeket a ZRuntime koordinál.

| Réteg | Elsődleges felelősség | Kulcsmodulok |
| :--- | :--- | :--- |
| Tudás és Betöltés | Relációs hármasok kinyerése és meglepetés-alapú memória kezelése. | crev_pipeline, surprise_memory |
| Alapgráf Motor | Az önhasonló relációs gráf és a temporális állapotok karbantartása. | nsir_core, temporal_graph |
| Optimalizálás és Következtetés | Gráfenergia minimalizálása sztochasztikus és hierarchikus módszerekkel. | esso_optimizer, reasoning_orchestrator |
| Végrehajtási Infrastruktúra | Hardver-absztrahált feldolgozás (GPU/VPU) és jelterjedés. | r_gpu, vpu, chaos_core |
| Ellenőrzés és Kvantum | Invariánsok formális bizonyítása és kvantumáramkör-végrehajtás. | formal_verification, quantum_logic, ibm_quantum |

Adatfolyam: A CREV Csővezetéktől az Ellenőrzött Kimenetig

Az információ az "Kinyerés-Optimalizálás-Ellenőrzés" mintát követve áramlik a rendszeren. A nyers bemenetet először a CREV (Relációs Tudáskinyerés) csővezeték dolgozza fel, amely strukturálatlan adatokat RelationalTriplet struktúrákká alakít.

1. Betöltés és Memória
A CREVPipeline egy többlépéses folyamatot alkalmaz: tokenizálás, hármas_kinyerés, érvényesítés, integráció és indexelés. A kinyert hármasokat a SurpriseMemoryManager szűri, amely a magas "meglepetési" (újdonság) értékű adatokat prioritizálja a hosszú távú megőrzéshez.

2. Gráffeldolgozás
A hármasok integrálódnak a SelfSimilarRelationalGraph-ba. A gráf minden csomópontja egy szemantikai entitást képvisel, amelyhez egy Qubit és phase érték tartozik. Az élek kapcsolatokat képviselnek specifikus EdgeQuality típusokkal (pl. entangled, coherent, fractal).

3. Optimalizálás és Végrehajtás
A ReasoningOrchestrator kezeli az EntangledStochasticSymmetryOptimizer-t (ESSO) a rendszer energiájának minimalizálásához. Ezt az optimalizálást a RelationalGraphProcessingUnit (RGPU) gyorsítja fel, amely egy 2D ProcessingCore egységekből álló hálót használ a párhuzamos gráfbejáráshoz.

4. Kvantumvégrehajtás és Ellenőrzés
Ha egy részgráf kvantumgyorsítást igényel, a QuantumTaskAdapter azonosítja a klasztereket és áramköröket küld a helyi RelationalQuantumLogic szimulátorhoz vagy az IBMQuantumClient-hez. Végül a FormalVerificationEngine biztosítja, hogy az eredményül kapott állapot kielégíti a gráfinvariánsokat.

Alapgráf Réteg (nsir_core)
A rendszer alapja a SelfSimilarRelationalGraph. Kezeli a Node és Edge életciklusokat.
- Kulcsfüggvény: Az applyHadamard és az applyPauliX közvetlenül manipulálják a csomópontok kvantumállapotát a gráfstruktúrán belül.
- Párhuzamosság: A hozzáférést egy Mutex közvetíti a GraphContext-en belül, hogy biztosítsa a szálbiztonságot a párhuzamos optimalizálás során.

Hardver Absztrakciós Réteg
A DANCING speciális feldolgozó egységekbe absztrahálja a fizikai hardvert:
- ChaosCoreKernel: Kezeli a ContentAddressableStorage-t (CAS) és a DynamicTaskScheduler-t. Kezeli az adatelhelyezést és a terheléselosztást.
- RGPU: Megvalósít egy AsynchronousNoC-ot (Network-on-Chip) az üzenetküldéshez a feldolgozó magok között, kifejezetten optimalizálva a gráfizomorfizmus-detektáláshoz.
- VPU: SIMD-gyorsított mátrixműveleteket és spektrális beágyazást biztosít a gráf vektorizálásához.

Végrehajtási Motorok

A rendszer két elsődleges végrehajtási útvonalat alkalmaz:
1. Klasszikus/Relációs Útvonal: Az RGPU és a ChaosCoreKernel kezeli. A propagateStep műveletekre összpontosít, ahol a jelek az élek mentén mozognak a weight és a quantum_correlation alapján.
2. Kvantum Útvonal: A QuantumTaskAdapter kezeli. Klaszterekbe csoportosítja a gráfot, ahol az entanglement_degree meghalad egy küszöbértéket, majd ezeket QuantumCircuit-ként hajtja végre.

Optimalizálási Hurok
A ReasoningOrchestrator visszacsatolási hurkot tart fenn:
1. Energiaszámítás: A calculateGraphEnergy kiszámítása.
2. Perturbáció: Az ESSO olyan lépéstípusokat alkalmaz, mint az edge_weight_shift vagy az entanglement_toggle.
3. Hűtés: A temperature beállítása adaptív hűtéssel.
4. Ellenőrzés: A FormalVerificationEngine érvényesíti, hogy az új állapot nem sérti az InvariantRegistry predikátumait.

---

1.2. ELSŐ LÉPÉSEK ÉS MODULTÉRKÉP

Ez az oldal technikai bevezető útmutatót nyújt a DANCING kódbázisba belépő mérnökök számára. Lefedi a fordítási rendszert, a központi modulregisztert, a C-kompatibilis belépési pontot és a fájlstruktúra átfogó leképezését a rendszer funkcionális alrendszereire.

A Projekt Fordítása

A DANCING keretrendszer Zig-ben íródott és a Zig fordítási rendszert használja. A projekt fordításához győződjön meg arról, hogy a Zig fordítóprogram (0.11.0 vagy újabb) telepítve van.

1. Szabványos fordítás: zig build
2. C API Megosztott Könyvtár: A c_api.zig fájl megosztott könyvtárként (libjaide) exportálható más nyelvekkel való integrációhoz.
3. Optimalizálási Szintek: Használja a -Doptimize=ReleaseSafe kapcsolót a futásidejű biztonsági ellenőrzésekkel rendelkező, éles környezetbe kész teljesítményhez, vagy a -Doptimize=ReleaseFast kapcsolót a végrehajtási motorok maximális átviteli sebességéhez.

Modulregiszter (mod.zig)

A mod.zig fájl a teljes kódbázis központi csomópontjaként szolgál. Importál minden alrendszert és újraexportálja a kulcstípusokat és függvényeket, egységes névteret biztosítva a belső és külső fogyasztók számára.

Centralizált Exporttérkép
A regiszter logikai rétegekbe szervezi a rendszert, az alapvető adatstruktúráktól a hardver absztrakcióig és az ellenőrzésig.

| Réteg | Elsődleges exportok | Forrásfájl |
| :--- | :--- | :--- |
| Alapgráf | SelfSimilarRelationalGraph, Node, Edge | mod.zig:3-3 |
| Kvantumlogika | RelationalQuantumLogic, QuantumCircuit | mod.zig:4-4 |
| Futtatókörnyezet | ZRuntime, ZVariable | mod.zig:5-5 |
| Végrehajtás | ChaosCoreKernel, RelationalGraphProcessingUnit | mod.zig:6-7 |
| Optimalizálás | EntangledStochasticSymmetryOptimizer | mod.zig:8-8 |
| Memória | ContentAddressableStorage, SurpriseMemoryManager | mod.zig:23-23 |
| Ellenőrzés | FormalVerificationEngine, TypeTheoryEngine | mod.zig:14-16 |

C API Belépési Pont (jaide_)

A külső integrációhoz a c_api.zig stabil C-kompatibilis interfészt biztosít. Átburkolja a Zig belső struktúrákat átlátszatlan kezelőkbe és szálbiztos hozzáférést biztosít a GraphContext mutex-en keresztül.

Modultérkép: Alrendszer Könyvtár

A következő táblázat minden .zig fájlt leképez a megfelelő alrendszerére és leírja elsődleges felelősségét a DANCING architektúrán belül.

| Fájl elérési útja | Alrendszer | Kulcsentitások |
| :--- | :--- | :--- |
| nsir_core.zig | Alapgráf | SelfSimilarRelationalGraph, Node, Edge |
| c_api.zig | Integráció | jaide_ függvények, CGraph, COptimizer |
| temporal_graph.zig | Alapgráf | TemporalGraph, GraphSnapshot, NodeVersion |
| esso_optimizer.zig | Optimalizálás | EntangledStochasticSymmetryOptimizer |
| reasoning_orchestrator.zig | Optimalizálás | ThoughtLevel, InferenceContext |
| fnds.zig | Optimalizálás | FNDSManager, FractalTree, FractalNodeData |
| chaos_core.zig | Végrehajtás | ChaosCoreKernel, DynamicTaskScheduler, CAS |
| r_gpu.zig | Végrehajtás | RelationalGraphProcessingUnit, AsynchronousNoC |
| vpu.zig | Végrehajtás | SimdVector, MatrixOps, LNSInstruction |
| signal_propagation.zig | Végrehajtás | SignalPropagationEngine, SignalState |
| crev_pipeline.zig | Tudás | CREVPipeline, RelationalTriplet, KnowledgeGraphIndex |
| z_runtime.zig | Tudás | ZRuntime, ZVariable, RelationalQuantumLogic |
| surprise_memory.zig | Tudás | SurpriseMemoryManager, SurpriseMetrics |
| quantum_logic.zig | Kvantum | QuantumState, QuantumCircuit, LogicGate |
| quantum_hardware.zig | Kvantum | QuantumBackend, IBMQuantumClient |
| quantum_task_adapter.zig | Kvantum | QuantumSubgraph, QASM Generálás |
| formal_verification.zig | Ellenőrzés | FormalVerificationEngine, TheoremProver |
| type_theory.zig | Ellenőrzés | TypeTheoryEngine, LinearTypeChecker |
| security_proofs.zig | Ellenőrzés | SecurityProofEngine, InformationFlowAnalysis |
| zk_verification.zig | Ellenőrzés | ZKInferenceProver, InferenceWitness |
| safety.zig | Segédprogramok | SafetyError, SecureRng, MonotonicClock |

Biztonság és Segédprogramok

A safety.zig modul biztosítja a keretrendszer megbízhatóságának alapját. Határellenőrzött típuskonverziókat és kriptográfiai primitíveket valósít meg, amelyeket minden rétegben használnak.

Kulcsfontosságú biztonsági entitások:
- SafetyError: Átfogó hibakészlet, amely lefedi a memória- és aritmetikai sértéseket.
- SecureRng: Hibrid entrópiaforrás, amely az std.crypto.random-ot egy Multiplikatív Kongruenciális Generátor (MCG) tartalékkal kombinálja.
- safeIntCast: Kimerítő túlcsordulás- és alulcsordulás-ellenőrzéseket végez az előjeles és előjel nélküli típusok között a konvertálás előtt.
- MonotonicClock: Nagy pontosságú időzítés a teljesítményprofilozáshoz és a temporális gráf verziókezeléséhez.

---

2. ALAPGRÁF MOTOR (nsir_core)

Az Alapgráf Motor a DANCING keretrendszer alapvető adatrétegeként szolgál. Megvalósítja a SelfSimilarRelationalGraph-ot, egy hibrid struktúrát, amely a klasszikus gráfelméletet kvantummechanikai tulajdonságokkal kombinálja. Ez a motor felelős a csomópontok és élek életciklusának kezeléséért, a kvantumkoherencia fenntartásáért a topológián keresztül, és a matematikai szubsztrátum biztosításáért mind a temporális verziókezeléshez, mind a C-kompatibilis együttműködési képességhez.

Rendszerarchitektúra Áttekintés

A motor áthidalja a nyers relációs adatok (Alany-Reláció-Tárgy hármasok) és a kvantumállapot-tenzorok közötti szakadékot. Komplex számokat foglal magában a kvantumamplitúdókhoz, metaadatokat kezel az önhasonlósághoz (fraktáldimenziók), és szálbiztos hozzáférést biztosít magas szintű kontextusokon keresztül.

Természetes Nyelv és Kódentitás Leképezés

A következő diagram bemutatja, hogyan képezhetők le a fogalmi relációs entitások specifikus Zig struktúrákra az nsir_core alrendszeren belül.

Természetes Nyelvi Tér:
- Alany/Entitás -> struct Node
- Kapcsolat/Predikátum -> struct Edge
- Állapot/Tulajdonság -> struct Qubit
- Edge -> enum EdgeQuality
- Node -> Qubit

Alapkomponensek

A motor három elsődleges absztrakcióra épül, amelyek meghatározzák a relációs modell állapotát és viselkedését:

1. Kvantummal Bővített Csomópontok: A hagyományos gráfcsomópontoktól eltérően az nsir_core minden Node-ja tartalmaz egy Qubit-et. Ez lehetővé teszi, hogy a csomópontok állapotok szuperpozíciójában létezzenek, képviselve a bizonytalanságot vagy a multimodális identitást.
2. Relációs Élek: Az élek meghatározzák a csomópontok közötti kapcsolatot és EdgeQuality jellemzi őket (pl. entangled, coherent, fractal). A quantum_correlation-t komplex számként tárolják az entitások közötti összefonódás modellezéséhez.
3. Önhasonló Relációs Gráf: A legfelső szintű SelfSimilarRelationalGraph tároló nagy teljesítményű hash-térképekkel kezeli ezeket az entitásokat.

Kulcs Alrendszerek

Az Alapgráf Motor három speciális területre van felosztva, amelyeket a következő gyermeklapok részleteznek:

SelfSimilarRelationalGraph: Csomópontok, Élek és Kvantumállapotok
Ez a modul kezeli a gráf alacsony szintű mechanikáját. Tartalmazza a Qubit struktúra megvalósítását, amely támogatja a normalizálást és a valószínűségszámítást. Kezeli a kvantumkapuk (Hadamard, Pauli-X/Y/Z) alkalmazását a csomópontokra és az összefonódás létrehozását csomópontpárok között.

C API (jaide_ Interfész)
A külső nyelvekkel és eszközökkel való integráció megkönnyítése érdekében a motor robusztus C-kompatibilis interfészt biztosít, amelyet a c_api.zig definiál. Ez a réteg jaide_ előtaggal exportál függvényeket, lehetővé téve a gráfmanipulációt, optimalizálást és kvantumállapot-mérést átlátszatlan kezelőkön (CGraph és COptimizer) keresztül.

Temporális Gráf és Verzionált Állapot
A relációs adatok gyakran időérzékenyek. A temporal_graph.zig modul kiterjeszti az alapgráfot verziókezelési képességekkel. Lehetővé teszi a rendszer számára a TemporalNode és TemporalEdge állapotok időbeli nyomon követését, pillanatfelvételek és történeti lekérdezések támogatásával egy InferenceContext-en keresztül.

Adatstruktúrák Összefoglalása

| Struktúra | Cél | Kulcsmezők |
| :--- | :--- | :--- |
| Qubit | Egy csomópont kvantumállapotát képviseli | a, b (Complex f64) |
| Node | A gráf alapvető entitása | id, data, qubit, phase |
| Edge | Entitások közötti kapcsolat | source, target, quality, weight |
| SelfSimilarRelationalGraph | Fő gráftároló | nodes, edges, allocator, mutex |

---

2.1. SELFSIMILARRELATIONALGRAPH: CSOMÓPONTOK, ÉLEK ÉS KVANTUMÁLLAPOTOK

A SelfSimilarRelationalGraph a DANCING keretrendszer alapvető adatstruktúrája, amely az nsir_core.zig-ben található. Hibrid relációs-kvantum modellt valósít meg, ahol a hagyományos gráftopológiát kvantumállapot-vektorokkal (qubitek), fázisadatokkal és fraktáldimenziókkal egészítik ki. Ez a motor kezeli a csomópontok és élek életciklusát, miközben matematikai primitíveket biztosít a kvantumkapu-alkalmazáshoz, az összefonódáshoz és az információkódoláshoz.

Alapvető Adatentitások

A gráf négy elsődleges típusból áll, amelyek meghatározzák állapotát és kapcsolatait.

| Típus | Szerep | Kulcsattribútumok |
| :--- | :--- | :--- |
| Qubit | Egyqubites kvantumállapotot képvisel. | a és b amplitúdók (Complex f64). |
| Node | Adatot és kvantumállapotot tartalmazó relációs entitás. | id, data, qubit, phase és metadata. |
| Edge | Irányított kapcsolat csomópontok között kvantumtulajdonságokkal. | source, target, quality, weight, quantum_correlation. |
| TwoQubit | 4-dimenziós állapot összefonódott párokhoz. | amplitudes (4 Complex f64 tömbje). |

Élminőség
Az élek EdgeQuality enum szerint vannak kategorizálva, amely meghatározza, hogyan kezeli a kapcsolatot az optimalizálási és terjedési motor:
- superposition: A kapcsolat valószínűségi állapotban létezik.
- entangled: A csomópontok nem-lokális kvantumkorrelációt osztanak meg.
- coherent: A csomópontok stabil fáziskapcsolatot tartanak fenn.
- collapsed: Klasszikus, determinisztikus kapcsolat.
- fractal: Önhasonló kapcsolat hierarchikus skálákon keresztül.

Gráf Életciklus és Kezelés

A SelfSimilarRelationalGraph struktúra kezeli a memóriafoglalást és a szálbiztonságot a teljes topológia számára.

- Inicializálás: Az init(allocator) inicializálja a belső StringHashMap-et a csomópontokhoz és az ArrayList-et az élekhez, miközben beállít egy Mutex-et a szálbiztos műveletekhez.
- Csomópont/Él Hozzáadása: Az addNode és az addEdge mély másolatokat készítenek a bemeneti adatokról és azonosítókról, hogy biztosítsák, hogy a gráf saját memóriával rendelkezzen.
- Tisztítás: A deinit() végigiterál az összes csomóponton és élen, felszabadítva a duplikált karakterláncokat és metaadat-térképeket a memóriaszivárgás megelőzése érdekében.

Kvantumműveletek és Kapuk

A gráf egyqubites és kétqubites műveleteket támogat, lehetővé téve a kvantumlogika szimulálását közvetlenül a relációs struktúrán.

Qubit Normalizálás
Minden Qubit normalizált, így |a|^2 + |b|^2 = 1. A normalizeInPlace függvény kezeli a NaN vagy nulla normájú szélső eseteket, alapértelmezés szerint a |0> bázisra visszaállva.

Kapu Alkalmazás
A gráf szabványos Pauli és Hadamard kapukat biztosít:
- Hadamard (H): Szuperpozíciót hoz létre a bázisállapotok transzformálásával.
- Pauli-X (NOT): Felcseréli a |0> és |1> amplitúdóit.
- Pauli-Y/Z: Fáziseltolásokat és komplex forgatásokat alkalmaz.

Összefonódás és Mérés
Az összefonódás két csomópont között az entangleNodes(id1, id2) segítségével jön létre, amely létrehoz egy TwoQubit állapotot (általában Bell-állapot) és frissíti a megfelelő EdgeQuality-t .entangled értékre. A mérés (measureNode) összeomlasztja a Qubit állapotot egy klasszikus bázisba a |β|^2 valószínűség alapján, hatékonyan "összeomlasztva" a csomópont kvantumbizonytalanságát.

Topológia Hashelés és Tenzorok

A gyors összehasonlítás és optimalizálás megkönnyítése érdekében a gráf kompakt hashbe vagy numerikus tenzorrá szerializálható.

- Topológia Hashelés: A computeTopologyHash függvény determinisztikus sorrendben végigiterál az összes csomóponton és élen, azonosítóikat, súlyaikat és kvantumállapotaikat egy SHA-256 hashelőbe táplálva. Ez egyedi ujjlenyomatot állít elő a gráf aktuális állapotáról.
- Tenzor Export: Az exportToTensor a relációs struktúrát lapos f32 tömbbé (VPU-kompatibilis) alakítja. Kódolja a csomópontfázisokat és az élsúlyokat, lehetővé téve a gráf feldolgozását szabványos gépi tanulási kernelekkel.

Kódolási és Dekódolási Logika
Az információ az encodeInformation segítségével injektálható a gráfba. Ez a függvény egy bájtszeletet vesz és elosztja azt a csomópontadatok vagy kvantumfázisok között. Ezzel szemben a decodeInformation rekonstruálja az adatokat a gráftopológia bejárásával és a tárolt attribútumok olvasásával, hidat biztosítva a klasszikus bitfolyamok és a relációs-kvantum állapot között.

---

2.2. C API (jaide_ INTERFÉSZ)

A jaide_ Interfész C-kompatibilis határt biztosít a DANCING keretrendszer számára, lehetővé téve a külső nyelvek és rendszerek számára, hogy interakcióba lépjenek a nagy teljesítményű Zig maggal. Átlátszatlan kezelők mögé absztrahálja a komplex SelfSimilarRelationalGraph-ot és az EntangledStochasticSymmetryOptimizer-t, biztosítva a memóriabiztonságot és a szálszinkronizálást egy dedikált GraphContext-en keresztül.

Rendszerarchitektúra és Adatfolyam

A C API hídként működik a "Természetes Nyelv/Külső Tér" és a DANCING mag "Kódentitás Tere" között. Szabványos C típusokat fordít belső Zig struktúrákká, miközben szálbiztos kontextust kezel a gráfmanipulációkhoz.

Alapvető Adatstruktúrák

Az interfész két elsődleges átlátszatlan kezelőre és egy kontextusstruktúrára támaszkodik, amely kezeli a párhuzamosságot.

Átlátszatlan Kezelők:
- CGraph: Mutató egy belső GraphContext-re. Minden gráfhoz kapcsolódó művelethez használják (csomópontok, élek hozzáadása, kapuk alkalmazása).
- COptimizer: Mutató az EntangledStochasticSymmetryOptimizer-re. Kezeli a sztochasztikus hőkezelési folyamatok életciklusát.

GraphContext
A többszálú környezetben való szálbiztonság érdekében a C API egy GraphContext-be csomagolja az alapgráfot. Ez a struktúra tartalmazza:
- inner: A tényleges SelfSimilarRelationalGraph példány.
- lock: Egy std.Thread.Mutex az exportált függvényhívások során való hozzáférés szinkronizálásához.
- allocator: A gráfműveletekhez használt memóriafoglaló.

C-Kompatibilis Típusok
Az API specifikus extern struktúrákat exportál az ABI-kompatibilitás biztosítása érdekében:
- CQuantumState: Komplex számot képvisel real és imag mezőkkel (f64).
- GateType: Egy enum, amely C egész számokat kvantumkapukra képez le: Identity (0), Hadamard (1), PauliX (2), PauliY (3), PauliZ (4).

Exportált Függvények (jaide_)

Gráfkezelés
Az API függvényeket biztosít a relációs gráf életciklusához és alapvető manipulációjához.

| Függvény | Leírás |
| :--- | :--- |
| jaide_create_graph | Lefoglal egy új GraphContext-et és inicializálja a SelfSimilarRelationalGraph-ot. |
| jaide_destroy_graph | Biztonságosan deinicializálja a gráfot és felszabadítja a kontextus memóriáját. |
| jaide_add_node | Új csomópontot szúr be kezdeti kvantumállapottal. JAIDE_ERROR_DUPLICATE_NODE-ot ad vissza, ha az azonosító már létezik. |
| jaide_add_edge | Súlyozott élt hoz létre két csomópont között megadott EdgeQuality-vel. |

Kvantumműveletek
A kvantumkapukat a gráf csomópontjaira alkalmazzák olyan burkolókon keresztül, amelyek kezelik a komplex szám konverzióját és a zárolást.

Optimalizálás: ESSO Integráció
Az EntangledStochasticSymmetryOptimizer (ESSO) a jaide_optimize függvénycsaládon keresztül van kitéve. Szimulált hőkezelést használ a gráf energiaállapotának minimalizálásához.

- Inicializálás: A jaide_create_optimizer paramétereket vesz fel az initial_temp, cooling_rate és max_iterations értékekhez.
- Konfiguráció: A jaide_optimizer_set_config lehetővé teszi a perturbációs valószínűségek (pl. perturb_edge_prob) futásidejű hangolását.
- Végrehajtás: Az optimalizáló végigiterál a lépéseken (élsúly-eltolások, csomópontfázis-változások) és rögzíti a sikert az OptimizationStatistics-ban.

Hibakezelés

Az API szabványos c_int visszatérési mintát használ a hibajelentéshez.

| Konstans | Érték | Jelentés |
| :--- | :--- | :--- |
| JAIDE_SUCCESS | 0 | A művelet sikeresen befejeződött. |
| JAIDE_ERROR_NULL_POINTER | -1 | Egy szükséges mutatóargumentum null volt. |
| JAIDE_ERROR_NODE_NOT_FOUND | -3 | A megadott csomópont-azonosító nem létezik a gráfban. |
| JAIDE_ERROR_THREADING | -16 | Nem sikerült megszerezni vagy felszabadítani a kontextus mutex-et. |
| JAIDE_ERROR_UNKNOWN_GATE | -17 | A megadott GateType nem támogatott. |

Megvalósítási Részletek

Szálbiztonság
Minden jaide_ függvénynek, amely módosítja vagy olvassa a gráfot, meg kell szereznie a GraphContext.lock-ot. Ez biztosítja, hogy még ha több szál is hívja a C API-t (pl. Python burkolóból vagy párhuzamos C++ alkalmazásból), az alapul szolgáló SelfSimilarRelationalGraph konzisztens állapotban maradjon.

Memóriakezelés
A C API felelős a GraphContext lefoglalásáért az inicializálás során megadott Zig Allocator segítségével. A külső hívóknak meg kell hívniuk a jaide_destroy_graph-ot a memóriaszivárgás megelőzése érdekében, mivel a Zig szemétgyűjtő (ha használják) nem követi nyomon ezeket az átlátszatlan kezelőket.

---

2.3. TEMPORÁLIS GRÁF ÉS VERZIONÁLT ÁLLAPOT

A temporal_graph.zig modul infrastruktúrát biztosít a gráfállapotok időbeli kezeléséhez. A statikus gráftól eltérően a Temporális Gráf minden csomópont és él lineáris verzióelőzményét tartja fenn, lehetővé téve az időponthoz kötött pillanatfelvételeket, a történeti auditálást és a temporális lekérdezéseket. Ez a rendszer kritikus fontosságú a DANCING keretrendszer azon képességéhez, hogy visszalépjen az optimalizálás során és elemezze a kvantum-relációs állapotok fejlődését.

Alapvető Verziókezelési Entitások

A rendszer az időt diszkrét Timestamp értékek sorozataként kezeli (nanoszekundumokban). Minden csomópont- vagy élváltozás egy új verzionált bejegyzést eredményez.

| Entitás | Kódszimbólum | Leírás |
| :--- | :--- | :--- |
| Időbélyeg | Timestamp | Egy i64, amely nanoszekundumokban képviseli az időt. |
| Csomópontverzió | NodeVersion | Tárolja a QuantumState-et, tulajdonságokat és egy verziószámlálót egy csomóponthoz egy adott időpontban. |
| Élverzió | EdgeVersion | Tárolja a súlyt, az EdgeQuality-t és metaadatokat egy élhez egy adott időpontban. |
| Temporális Csomópont | TemporalNode | Egy tároló az összes NodeVersion bejegyzéshez, amelyek egy adott csomópont-azonosítóhoz tartoznak. |
| Temporális Él | TemporalEdge | Egy tároló az összes EdgeVersion bejegyzéshez, amelyek egy adott EdgeKey-hez tartoznak. |

Adatfolyam: Verzió Létrehozása
Amikor egy csomópont állapota frissül a TemporalGraph.updateNode segítségével, a rendszer nem írja felül a meglévő adatokat. Ehelyett létrehoz egy új NodeVersion-t, növeli a verziószámlálót és hozzáfűzi a TemporalNode.versions listához.

Gráf Pillanatfelvételek és Visszaállítás

A GraphSnapshot a teljes gráf konzisztens nézetét képviseli egy adott Timestamp-nél. Rögzíti az összes csomópont és él állapotát, ahogyan azok az adott időpontban vagy közvetlenül előtte léteztek.

Pillanatfelvétel Mechanikája:
1. Létrehozás: A TemporalGraph.createSnapshot függvény végigiterál az összes csomóponton és élen, megkeresve a legújabb verziót, amelynek időbélyege kisebb vagy egyenlő a célnál.
2. Visszaállítás: A TemporalGraph.restoreFromSnapshot visszaállítja a gráf aktuális "aktív" állapotát a pillanatfelvételben tárolt értékekre.
3. Auditnapló: Minden jelentős változás vagy pillanatfelvétel-művelet rögzítésre kerül a HistoryEntry naplóban a törvényszéki elemzéshez.

Temporális Lekérdezés és Szűrés

A TemporalQuery motor lehetővé teszi a gráfelemek összetett lekérését temporális korlátok és egyéni predikátumok alapján.

- TemporalQuery Struktúra: Meghatározza a keresési ablakot (start_time-tól end_time-ig) és opcionális szűrőket.
- Szűrés: A felhasználók NodeFilterFn és EdgeFilterFn függvényeket adhatnak meg az eredmények szűkítéséhez kvantumállapot, élminőség vagy tulajdonságok alapján.

Végrehajtási Logika
A TemporalGraph.queryNodes és queryEdges függvények a következőket hajtják végre:
1. Végigiterálnak a TemporalNode vagy TemporalEdge gyűjteményeken.
2. Minden entitásnál végigiterálnak a versions listán.
3. Alkalmazzák az időbélyeg-tartomány ellenőrzést.
4. Végrehajtják a megadott FilterFn-t a verzióadatokkal szemben.
5. Visszaadnak egy ArrayList-et az egyező verziókkal.

InferenceContext és Nagy Frekvenciájú Frissítések

A nagy frekvenciájú frissítésekhez (pl. kvantumszimulációk vagy valós idejű következtetés során) az InferenceContext speciális környezetet biztosít az átmeneti állapotok kezeléséhez, mielőtt azok a permanens temporális előzményekbe kerülnének.

Kulcskomponensek:
- Érvényességi Tartományok: Az InferenceContext-ben lévő éleknek specifikus érvényességi időszakai lehetnek, amelyeket a start_timestamp és az end_timestamp határoz meg.
- InferenceEdge: Kiterjeszti a szabványos élmodellt temporális érvényességgel, elsősorban az InferenceContext-en belül használják.
- Kontextuskezelés: Az InferenceContext saját csomópont- és éltérképet tart fenn, "vázlatlapként" működve a nagy frekvenciájú frissítésekhez.

Megvalósítási Részletek: Él Érvényessége

A temporális gráf élei explicit érvényességi tartományokat támogatnak, lehetővé téve a rendszer számára, hogy olyan kapcsolatokat modellezzen, amelyek csak meghatározott intervallumokban léteznek.

- Él Érvényessége: A TemporalEdge nyomon követi a first_seen és last_seen időbélyegeket.
- Jelenlét Ellenőrzés: A TemporalEdge.existsAt(timestamp) meghatározza, hogy egy él aktív volt-e egy adott időpontban, ellenőrizve, hogy az időbélyeg az ismert verzióelőzményein belül esik-e.
- Súlyevolúció: Mivel az EdgeVersion tárolja a súlyokat, a rendszer kiszámíthatja egy él averageWeight értékét egy adott temporális ablakban.

---

3. OPTIMALIZÁLÁSI MOTOROK

A DANCING keretrendszer többszintű optimalizálási architektúrát alkalmaz, amelyet a SelfSimilarRelationalGraph energiaállapotának minimalizálására terveztek. Ez a folyamat magában foglalja a kvantumkoherencia, a relációs kapcsolat és a fraktálstruktúra integritásának egyensúlyozását. A rendszer sztochasztikus helyi keresések és magas szintű hierarchikus következtetés között vált, hogy elkerülje a helyi minimumokat és globális szimmetriákat fedezzen fel.

Architektúra Áttekintés

Az optimalizálási verem három elsődleges alrendszerből áll, amelyek kölcsönhatásba lépnek a gráfállapot finomítása érdekében:

1. Összefonódott Sztochasztikus Szimmetria Optimalizáló (ESSO): Az alacsony szintű végrehajtási motor, amely atomi mutációkat hajt végre a gráfon szimulált hőkezelés segítségével.
2. Érvelési Orchestrátor: Magas szintű vezérlő, amely kezeli az optimalizálási életciklust több ThoughtLevel szinten (Helyi, Globális, Meta).
3. Fraktál Csomópont Adatrendszer (FNDS): Speciális adatindexelési és -lekérési rendszer, amely fenntartja a gráf önhasonló tulajdonságait és strukturális metrikákat biztosít az objektív függvényekhez.

Összefonódott Sztochasztikus Szimmetria Optimalizáló (ESSO)

Az ESSO az esso_optimizer.zig-ben definiált alapvető sztochasztikus motor. Kifinomult szimulált hőkezelési algoritmust valósít meg, amely figyelembe veszi mind a klasszikus gráftopológiát, mind a kvantum-összefonódási állapotokat.

- Lépéstípusok: Az optimalizáló hét különböző lépéstípust alkalmaz a gráf perturbálásához, beleértve az edge_weight, node_phase, entanglement és fractal_dimension típusokat.
- Szimmetria Detektálás: Geometriai és relációs szimmetriákat azonosít (tükrözés, forgatás, eltolás) az optimalizálási tér egyszerűsítéséhez.
- Visszafordíthatóság: Minden lépés rögzítésre kerül egy UndoLog-ban, lehetővé téve a rendszer számára, hogy visszatérjen az előző állapotokhoz, ha egy lépés a hűtési küszöb fölé növeli a globális energiát.

Érvelési Orchestrátor

Az Érvelési Orchestrátor az optimalizálási folyamat felügyelőjeként működik, ahogyan azt a reasoning_orchestrator.zig definiálja. Háromszintű hierarchiába szervezi az ESSO munkáját a konvergencia biztosítása érdekében.

| Szint | Név | Fókusz |
| :--- | :--- | :--- |
| 0 | Helyi | Kis léptékű perturbációk és helyi kapcsolat. |
| 1 | Globális | Szimmetria detektálás és globális energiaminimalizálás. |
| 2 | Meta | Fázisátmenetek felügyelete és konvergencia detektálás. |

Az orchestrátor kiszámítja a modulation_factor-t, amely skálázza a különböző objektív függvények befolyását az aktuális ReasoningPhase alapján.

Fraktál Csomópont Adatrendszer (FNDS)

Az FNDS biztosítja az önhasonló optimalizálás strukturális alapját. Az fnds.zig-ben található, kezeli a gráfcsomópontok rekurzív tulajdonságait.

- FractalTree: Speciális indexelési struktúra, amely támogatja a többskálás keresést és bejárást.
- Integritás: A fractal_signature és a FractalNodeData segítségével biztosítja, hogy ahogy a gráf növekszik vagy optimalizálódik, önhasonló tulajdonságai konzisztensek maradjanak.
- Mintafelfedezés: A SelfSimilarIndex segíti az ESSO-t az ismétlődő motívumok azonosításában, amelyek egyetlen egységként optimalizálhatók.

Rendszerintegrációs Diagram

Ez a diagram összeköti a logikai optimalizálási koncepciókat a végrehajtásukért felelős specifikus Zig struktúrákkal és kezelőkkel.

Optimalizálási Komponens Leképezés:

ReasoningOrchestrator:
- ArrayList~ReasoningPhase~ phase_history
- OrchestratorStatistics statistics
- runOrchestration()

EntangledStochasticSymmetryOptimizer:
- OptimizationState state
- ArrayList~UndoLog~ undo_stack
- step()

FNDSManager:
- FractalTree tree
- CoalescedHashMap index
- calculateFractalDimension()

ReasoningOrchestrator --> EntangledStochasticSymmetryOptimizer: "kezeli az életciklust"
EntangledStochasticSymmetryOptimizer --> FNDSManager: "strukturális metrikákat kérdez le"
ReasoningOrchestrator ..> ReasoningPhase: "nyomon követi"

---

3.1. ÖSSZEFONÓDOTT SZTOCHASZTIKUS SZIMMETRIA OPTIMALIZÁLÓ (ESSO)

Az Összefonódott Sztochasztikus Szimmetria Optimalizáló (ESSO) egy nagy teljesítményű optimalizálási motor, amelyet a SelfSimilarRelationalGraph energiaállapotának minimalizálására terveztek, mind topológiai, mind kvantumállapot-konfigurációk feltárásával. Kifinomult szimulált hőkezelési életciklust valósít meg, amely integrálja a szimmetria detektálást, a fraktáldimenziót és a kvantum-összefonódást.

Alapvető Életciklus és Szimulált Hőkezelés

Az ESSO egy szimulált hőkezelési folyamaton keresztül működik, amelyet az OptimizationState kezel. Az optimalizáló iteratívan perturbálja a gráfállapotot, kiértékel egy objektív függvényt, és az aktuális "hőmérséklet" alapján elfogadja vagy elutasítja a változást.

Optimalizálási Állapot és Konfiguráció
Az OptimizationState struktúra fenntartja az optimalizálás futásidejű kontextusát, beleértve a célgráfot, az aktuális hőmérsékletet és a teljesítménymetrikákat.

| Mező | Leírás |
| :--- | :--- |
| graph | Az optimalizált SelfSimilarRelationalGraph. |
| temperature | Az aktuális termikus energia, amely szabályozza a lépések elfogadását. |
| best_energy | Az optimalizálás során talált legalacsonyabb energia (objektív érték). 
| cooling_rate | Az a tényező, amellyel a hőmérséklet csökken minden lépés után. |
| undo_log | UndoEntry objektumok vereme a visszafordítható mutációkhoz. |

Adatfolyam: Hőkezelési Lépés
A step függvény az optimalizáló egyetlen iterációját képviseli.

1. Kiválasztás: Egy mutációtípus sztochasztikusan kerül kiválasztásra.
2. Mutáció: A gráf módosul (pl. élsúly-beállítás).
3. Kiértékelés: Az ObjectiveFunction kiszámítja az új energiát.
4. Metropolis-kritérium: Ha az új energia alacsonyabb, a lépés elfogadásra kerül. Ha magasabb, P = e^(-ΔE/T) valószínűséggel fogadják el.
5. Visszaállítás: Ha elutasítják, az UndoLog segítségével visszaállítják az előző állapotot.

---

Hét Lépéstípus

Az ESSO hét különböző mutációtípust alkalmaz a relációs gráf nagy dimenziós keresési terének bejárásához.

| Lépéstípus | Megvalósítás | Leírás |
| :--- | :--- | :--- |
| Élsúly | mutateEdgeWeight | Véletlenszerűen perturbálja egy meglévő él súlyát egy meghatározott tartományon belül. |
| Csomópontfázis | mutateNodePhase | Beállítja egy csomópont kvantumállapotának phase komponensét. |
| Összefonódás | mutateEntanglement | Összefonódási kapcsolatokat hoz létre vagy old fel különböző csomópontok qubitjei között. |
| Szimmetria-transzformáció | mutateSymmetry | Affin transzformációkat (tükrözés, forgatás, eltolás) alkalmaz a csomópontkoordinátákra. |
| Qubit-amplitúdó | mutateQubitAmplitude | Módosítja egy csomópont belső Qubit-jének komplex amplitúdóit (α, β). |
| Fraktáldimenzió | mutateFractalDim | Beállítja a csomópontok fractal_dimension tulajdonságát az önhasonlóság optimalizálásához. |
| Él-kapcsoló | mutateEdgeToggle | Sztochasztikusan ad hozzá vagy távolít el éleket a topológiai kapcsolódás feltárásához. |

---

Visszafordíthatóság az UndoLog Segítségével

A lépések sztochasztikus elutasításának támogatásához anélkül, hogy klónozni kellene az egész gráfot (ami számítási szempontból drága), az ESSO egy UndoLog-ot használ. Minden mutáció rögzíti az inverz műveletet.

---

Szimmetria Detektálás és Transzformáció

Az ESSO a szimmetriát elsőrendű optimalizálási célként kezeli. A SymmetryGroup enum meghatározza a motor által támogatott geometriai műveleteket.

Szimmetriacsoportok:
- Identity: Nincs változás.
- Reflection: Tükrözés egy parameters[3] által meghatározott tengely mentén.
- Rotation: Rögzített lépések (90, 180, 270) vagy custom_rotation.
- Translation: Lineáris elmozdulás.

A SymmetryTransform struktúra kezeli ezeknek a műveleteknek a matematikáját egy Affine mátrix segítő segítségével. Az optimalizáló megpróbál olyan transzformációkat találni, amelyek minimalizálják a "Szimmetria Hibát" - a csomópontok közötti távolságot, miután egy csoportműveletet alkalmaztak.

---

Objektív Függvények

Az energiatájat az ObjectiveFunction határozza meg. Több beépített függvény súlyozható és kombinálható:

1. defaultGraphObjective: Kiegyensúlyozott heurisztika, amely figyelembe veszi a kapcsolódást és a stabilitást.
2. connectivityObjective: Minimalizálja az izolált klaszterek számát és optimalizálja az úthosszakat.
3. quantumCoherenceObjective: Jutalmazza a magas coherence_degree-vel és stabil összefonódással rendelkező állapotokat.
4. fractalDimensionObjective: A gráfot egy célzott fraktáldimenzió felé tereli, elősegítve az önhasonló struktúrákat.

---

Adaptív Hűtés és Újrafűtés

A helyi minimumok elkerülése érdekében az optimalizáló adaptív ütemezést alkalmaz:
- Hűtés: A hőmérséklet a cooling_rate-tel csökken minden sikeres lépés után.
- Újrafűtés: Ha az energia egy küszöbszámú iteráción át statikus marad (stagnálás), a hőmérséklet megugrasztódik, hogy a rendszer "kiugorhasson" egy helyi minimumból.

---

Következtetési Integráció

A modulateInferenceTensor függvény összeköti az optimalizálót a végrehajtási motorokkal (pl. a VPU-val). Az optimalizált gráfállapotot numerikus tenzor formátummá alakítja, amely alkalmas az InferenceContext számára.

- Súlymoduláció: Az élsúlyokat az EdgeQuality alapján skálázzák.
- Fázisinjektálás: A csomópontfázisok komplex értékekre képezhetők le a tenzorban.
- Szimmetria-igazítás: A tenzor úgy van strukturálva, hogy megőrizze a detektált SymmetryGroup tulajdonságokat.

---

3.2. ÉRVELÉSI ORCHESTRÁTOR

Az Érvelési Orchestrátor a DANCING keretrendszer magas szintű koordinációs motorja. Háromszintű hierarchikus érvelési folyamatot kezel, amelyet a SelfSimilarRelationalGraph energiaállapotának minimalizálására terveztek a helyi strukturális integritás, a globális szimmetria és a meta-szintű fázisfelügyelet egyensúlyozásával. Hídként működik az EntangledStochasticSymmetryOptimizer (ESSO) sztochasztikus optimalizálása és a ChaosCoreKernel hardver-tudatos feladatütemezése között.

Hierarchikus Érvelési Architektúra

Az orchestrátor három különböző ThoughtLevel szakaszon keresztül működik, amelyek mindegyike a relációs gráf különböző hatókörét célozza meg.

| Gondolati Szint | Hatókör | Fókusz | Megvalósítási Stratégia |
| :--- | :--- | :--- | :--- |
| local | Csomópont-szomszédságok | Helyi Perturbációk | Nagy frekvenciájú, kis késleltetésű frissítések a csomópontfázisokhoz és a közvetlen élsúlyokhoz. |
| global | Gráftopológia | Szimmetria és Káosz | Forgásos/eltolási szimmetriák detektálása és globális kapcsolódás-optimalizálás. |
| meta | Fázisfelügyelet | Konvergencia és Moduláció | Az energiadeltáknak magas szintű monitorozása és modulációs tényezők levezetése a ChaosCoreKernel számára. |

---

Alapvető Adatstruktúrák

ThoughtLevel Enum
Meghatározza az érvelési ciklus mélységét.
- local (0): Mikro-beállításokra összpontosít.
- global (1): Makro-strukturális mintákra összpontosít.
- meta (2): Magára az optimalizálási pályára összpontosít.

ReasoningPhase
Nyomon követi egy specifikus érvelési végrehajtási blokk állapotát és előrehaladását.
- inner_iterations: Sztochasztikus lépések száma egyetlen fázison belül.
- outer_iterations: A fázislogika ismétlésének száma.
- current_energy / previous_energy: Konvergencia-detektáláshoz használják.
- pattern_captures: A fázis során felfedezett szimmetriákat azonosító PatternId (32 bájtos hashek) listája.

---

Megvalósítási Részletek

Konvergencia Detektálás
Az orchestrátor meghatározza, hogy egy ReasoningPhase elért-e stabil állapotot a gráfenergia relatív változásának kiszámításával.

Delta = |Aktuális - Előző|
Nevező = max(|Előző|, 1.0)
(Delta / Nevező) < küszöbérték -> Konvergált

Gráfenergia és Moduláció
Az orchestrátor kölcsönhatásba lép a SelfSimilarRelationalGraph-fal az energiaállapotok kiszámításához. Ezeket az állapotokat egy modulation_factor levezetésére használják, amely befolyásolja, hogyan prioritizálja a ChaosCoreKernel az adatösszefonódást és a feladatütemezést.

Kulcsfüggvények a ReasoningOrchestrator-ban:
- init: Inicializálja az orchestrátort a gráfra, az ESSO optimalizálóra és a Chaos kernelre mutató hivatkozásokkal.
- setParameters: Konfigurálja a fast_inner_steps (alapértelmezett 50) és a slow_outer_steps (alapértelmezett 10) értékeket.
- setProcessingLimits: Korlátozza az egyetlen érvelési ciklusban perturbálható csomópontok vagy élek számát a számítási robbanás megelőzése érdekében.

Integráció az ESSO-val és a ChaosCoreKernel-lel
A ReasoningOrchestrator az "agy", amely vezérli az EntangledStochasticSymmetryOptimizer-t (ESSO). Míg az ESSO végzi a tényleges sztochasztikus lépéseket (pl. él-kapcsolók, fáziseltolások), az Orchestrátor dönti el, hogy melyik ThoughtLevel-t alkalmazza és mennyi ideig az OrchestratorStatistics alapján.

---

Statisztikák és Monitorozás
Az OrchestratorStatistics struktúra folyamatos nyilvántartást vezet a rendszer érvelési hatékonyságáról. Nyomon követi:
- Fázisszámok: A befejezett local, global és meta fázisok bontása.
- Konvergencia Idő: Mozgóátlag arról, hogy mennyi ideig tart a fázisoknak elérni a convergence_threshold-ot.
- Legjobb Energia: Az összes érvelési ciklus során elért legalacsonyabb energiaállapot.
- Felfedezett Minták: Az ESSO által azonosított és az orchestrátor által rögzített szimmetriaminták összesített száma.

---

3.3. FRAKTÁL CSOMÓPONT ADATRENDSZER (FNDS)

A Fraktál Csomópont Adatrendszer (FNDS) egy hierarchikus adatkezelési alrendszer az nsir_core-on belül, amely önhasonló adatstruktúrákat és mintafelfedezést kezel. Mechanizmust biztosít a gráfadatok több felbontási skálán való megjelenítéséhez, lehetővé téve a rendszer számára a fraktáldimenzió-számítások elvégzését és a strukturális integritás fenntartását kriptográfiai aláírások segítségével.

FNDS Kezelő és Architektúra

Az FNDSManager a fraktál adatműveletek központi orchestrátoraként működik. Kezeli a FractalTree példányok életciklusát, globális statisztikákat tart fenn, és gyorsítótárazási réteget biztosít a fraktálcsomópontokhoz való nagy frekvenciájú hozzáféréshez.

---

Alapvető Adatstruktúrák

FractalNodeData és Integritás
A FractalNodeData struktúra a fraktálhierarchia egyetlen egységét képviseli. Minden csomópont fenntart egy fractal_signature-t, egy SHA-256 hasht, amelyet a csomópont azonosítójából, adataiból, súlyából és skálájából vezetnek le. Ez biztosítja, hogy a hierarchiában bekövetkező bármilyen sérülés vagy jogosulatlan módosítás felismerhető legyen.

| Mező | Típus | Leírás |
| :--- | :--- | :--- |
| id | []const u8 | A csomópont egyedi azonosítója. |
| data | []const u8 | Nyers hasznos adat vagy relációs tartalom. |
| weight | f64 | Fontossági tényező a helyi szinten belül. |
| scale | f64 | A csomópont térbeli vagy felbontási skálája. |
| fractal_signature | [32]u8 | Kriptográfiai integritás hash. |

FractalLevel és Skálafaktorok
A csomópontok FractalLevel tárolókba vannak szervezve. Minden szint az önhasonló struktúra egy specifikus felbontását képviseli. A scale_factor meghatározza a részletesség arányát az aktuális szint és szülői/gyermekei között, ami kritikus fontosságú a gráf Hausdorff-szerű fraktáldimenziójának kiszámításához.

---

FractalTree Megvalósítás

A FractalTree egy kiegyensúlyozott hierarchikus struktúra, amelyet a fraktálcsomópontok hatékony beillesztéséhez, kereséséhez és bejárásához terveztek.

Kulcsműveletek:
1. Beillesztés: FractalNodeData bejegyzést ad hozzá. Ha az elágazási tényező meghaladódik, a fa újraegyensúlyozási műveletet hajt végre a maximális mélységkorlát fenntartásához.
2. Keresés: Többskálás keresést végez. Csomópontokat azonosítóval vagy specifikus fractal_signature töredékek egyeztetésével tud megtalálni.
3. Bejárás: Mélységi (DFS) és szélességi (BFS) keresést támogat a súlyok összesítéséhez szinteken keresztül az energiaszámításhoz.

Egyensúlyozás és Korlátok
A fa szigorú korlátokat érvényesít a branching_factor és a max_depth értékekre, hogy megakadályozza a degenerált hierarchiákat, amelyek érvénytelenítik a fraktáldimenzió-számításokat.

---

Mintafelfedezés és Indexelés

SelfSimilarIndex
A SelfSimilarIndex felelős az ismétlődő strukturális minták (motívumok) azonosításáért a fraktálfán belül. Egy CoalescedHashMap-et használ a mintahelyek hatékony tárolásához, minimalizálva a memóriafelhasználást a redundáns önhasonló struktúrák esetén.

CoalescedHashMap
Egy speciális hash-térkép megvalósítás, amelyet az indexelő használ az ütközések kezelésére oly módon, hogy megőrzi a fraktálminták térbeli lokalitását. Ez elengedhetetlen a ReasoningOrchestrator számára a globális szimmetriák azonosításához.

---

Numerikus Elemzés: Fraktáldimenzió

Az FNDS kiszámítja a gráf fraktáldimenzióját, hogy visszajelzést adjon az ESSO optimalizálónak. A számítás a következő logikát követi:
D = log(N) / log(1/s)
ahol N az önhasonló darabok száma (gyermekek száma) és s a skálafaktor.

Statisztikák és Telemetria
Az FNDSStatistics struktúra nyomon követi a rendszer állapotát:
- Átlagos Fa Mélység: Jelzi, ha a hierarchia túl méllyé válik (nem hatékony).
- Gyorsítótár Találati Arány: Monitorozza az LRUCache hatékonyságát a fraktálcsomópontokhoz.
- Mintasűrűség: A total_patterns_indexed segítségével mérve.

| Metrika | Kódentitás |
| :--- | :--- |
| Memóriahasználat | FNDSStatistics.memory_used |
| Gyorsítótár Teljesítmény | FNDSStatistics.cache_hit_ratio |
| Fa Mélység | FNDSStatistics.average_tree_depth |

---

Segédkomponensek

LRUCache
Az LRUCache biztosítja, hogy a gyakran hozzáfért FractalNodeData memóriában maradjon, csökkentve a SHA-256 aláírás-ellenőrzés terhelését a bejárás során.

FractalEdgeData
A fraktálrendszeren belüli élek EdgeType változatokon keresztül vannak típusozva, lehetővé téve a rendszer számára a megkülönböztetést:
- Hierarchikus Élek: Szülőcsomópontokat gyermekcsomópontokhoz kapcsolnak.
- Keresztskálás Élek: Különböző skálákon lévő csomópontokat kapcsolnak össze, amelyek szemantikai hasonlóságot osztanak meg.
- Laterális Élek: Szabványos relációs kapcsolatok ugyanazon a FractalLevel-en belül.

---

4.1. CHAOSCOREKERNEL ÉS TARTALOM-CÍMEZHETŐ TÁROLÁS

A ChaosCoreKernel a DANCING keretrendszer alapvető végrehajtási és memóriakezelési rétege. Speciális környezetet biztosít a nagy frekvenciájú gráfműveletek kezeléséhez Tartalom-Címezhető Tárolás (CAS), lokalitás-tudatos feladatütemező és adatösszefonódási mechanizmusok segítségével, amelyek megkönnyítik a gyors következtetést.

1. Tartalom-Címezhető Tárolás (CAS)

A ContentAddressableStorage (CAS) rendszer a memóriát diszkrét MemoryBlock egységekként kezeli, ahol az adatokat tartalom-alapú hashek indexelik, nem pedig tetszőleges címek. Ez automatikus deduplikációt biztosít és mechanizmust nyújt az adatok életciklusának nyomon követéséhez hozzáférési minták segítségével.

Kulcsstruktúrák:
- MemoryBlock: A tárolás atomi egysége, amely nyers adatokat, a content_hash-t és metaadatokat tartalmaz az ütemezéshez és kiürítéshez.
- MemoryBlockState: Nyomon követi egy blokk életciklusát az állapotokon keresztül: free, allocated, entangled vagy migrating.
- ContentAddressableStorage: Az elsődleges tároló, amely kezeli a BLOCK_ID-tól a MemoryBlock-ig való leképezést.

CAS Műveletek
A CAS módszereket biztosít az adatblokkok tárolásához, lekéréséhez és életciklusának kezeléséhez:
- store: Deduplikálja a bejövő adatokat a content_hash ellenőrzésével a foglalás előtt.
- get: Lekér egy blokkot és frissíti az access_count és last_access_time értékeket a kiürítési algoritmus számára.
- evictLowPriorityBlocks: Eltávolítja a blokkokat az access_count, last_access_time és az entangled állapot kombinációja alapján.

2. DynamicTaskScheduler és Adatlokalitás

A DynamicTaskScheduler kezeli a TaskDescriptor objektumok végrehajtását több ProcessingCore egységen keresztül. Prioritizálja az adatlokalitást a magok közötti kommunikációs terhelés minimalizálásához.

Feladatütemezési Logika
Az ütemező Adatlokalitás-Pontozási mechanizmust használ a feladatok magokhoz rendeléséhez. Kiértékeli, hogy egy feladat data_dependencies-ei jelenleg hol találhatók a memóriában.

| Függvény | Leírás |
| :--- | :--- |
| scheduleTask | Egy TaskDescriptor-t rendel a legalkalmasabb maghoz a munkaterhelés és az adataffinitás alapján. |
| calculateLocalityScore | Pontszámot számít (0.0 - 1.0) a szükséges MemoryBlock-ok százaléka alapján, amelyek már jelen vannak egy mag helyi gyorsítótárában. |
| balanceLoad | Időszakosan újraosztja a feladatokat, ha egy mag terhelése meghaladja a LOAD_HIGH_THRESHOLD értéket. |

3. ProcessingCore és Munkaterhelési Metrikák

A ProcessingCore struktúra egy logikai végrehajtási egységet képvisel. Valós idejű telemetriát követ nyomon a DynamicTaskScheduler tájékoztatásához.

- Munkaterhelési Metrikák: Minden mag fenntartja a current_load, cycle_count és throughput_bps értékeket.
- Erőforrás-nyomon Követés: A ProcessingCore.updateMetrics() segítségével monitorozva, amelyet minden feladat befejezése után hívnak meg a mag jövőbeli ütemezési elérhetőségének beállításához.

4. Adatösszefonódás és Elhelyezési Optimalizálás

Az adatösszefonódás a ChaosCoreKernel-ben a MemoryBlock-ok logikai összekapcsolását jelenti, amelyeket gyakran együtt érnek el. Ezt a DataFlowAnalyzer kezeli.

Összefonódási Mechanizmus
Amikor a DataFlowAnalyzer magas temporális korrelációt észlel két BlockId között, meghívja az addEntangledBlock-ot. Ez a metaadat tájékoztatja az optimizeDataPlacement függvényt, hogy ezeket a blokkokat ugyanazon a ProcessingCore-on vagy szomszédos memóriacsomópontokon tartsa.

5. InferenceHooks Burkoló

Az InferenceHooks struktúra visszahívások készletét biztosítja, amelyek integrálják a kernelt a szélesebb DANCING ökoszisztémával (pl. crev_pipeline.zig vagy signal_propagation.zig).

- onDataAccess: Akkor aktiválódik, amikor egy MemoryBlock lekérésre kerül, lehetővé téve a DataFlowAnalyzer számára a hozzáférési minták frissítését.
- onTaskComplete: Jelzi a DynamicTaskScheduler-nek, hogy frissítse a mag terhelési metrikáit és potenciálisan aktiválja a balanceLoad-ot.
- onEntanglementDetected: Lehetővé teszi a rendszer számára, hogy reagáljon az új logikai összekapcsolásokra az adatelhelyezés újraoptimalizálásával.

6. Konfigurációs Konstansok Összefoglalása

A kernel viselkedését a ChaosCoreConfig irányítja:

| Konstans | Érték | Cél |
| :--- | :--- | :--- |
| BLOCK_ID_SIZE | 16 | A memóriablock egyedi azonosítójának mérete. |
| OPTIMIZATION_THRESHOLD | 0.6 | Az adatelhelyezési optimalizálás aktiválásának küszöbértéke. |
| LOAD_HIGH_THRESHOLD | 1.3 | Az a terhelési szint, amelynél egy mag túlterheltnek minősül. |
| BALANCE_INTERVAL_CYCLES | 100 | A terheléselosztási rutin gyakorisága. |

---

4.2. RELÁCIÓS GRÁF FELDOLGOZÓ EGYSÉG (RGPU)

A Relációs Gráf Feldolgozó Egység (RGPU) egy speciális, hardver-absztrahált végrehajtási réteg, amelyet párhuzamos gráffeldolgozáshoz, izomorfizmus-detektáláshoz és dinamikus élsúly-optimalizáláshoz terveztek. Egy 2D hálót valósít meg feldolgozó magokból, amelyek egy Aszinkron Chip-hálózaton (NoC) keresztül kapcsolódnak egymáshoz.

1. Architektúra Áttekintés

Az RGPU architektúrát az RGPU struktúra határozza meg, amely ProcessingCore egységek gyűjteményét koordinálja. A rendszer 2D háló topológiaként működik, ahol minden mag a SelfSimilarRelationalGraph egy helyi töredékét tárolhatja.

2D Háló és Feldolgozó Magok
Minden ProcessingCore fenntartja saját állapotát, helyi gráftárolóját és teljesítménymetrikáit.
- Állapotok: A magok az idle, processing, communicating és power_gated állapotok között váltanak.
- Helyi Gráf: A magok vagy saját helyi gráffal rendelkeznek, vagy egy megosztottra hivatkoznak.
- Munkaterhelés-nyomon Követés: A getWorkload függvény kiszámítja a mag kihasználtságát az aktív és tétlen ciklusok alapján.

Chip-hálózat (NoC)
A magok közötti kommunikációt az AsynchronousNoC kezeli, amely XY útválasztást és prioritás-alapú üzenetkézbesítési rendszert valósít meg.
- Útválasztás: A routeMessage függvény kiszámítja a Manhattan-távolságot és meghatározza a következő ugrást a 2D hálóban.
- Prioritási Sor: Az üzenetek egy PriorityQueue-ban tárolódnak, a priority mező és a timestamp alapján rendezve.
- Üzenettípusok: A támogatott protokollok közé tartozik a weight_update, graph_sync, isomorphism_result és power_control.

2. Gráfelosztás és Szinkronizálás

Az RGPU egy globális SelfSimilarRelationalGraph-ot oszt el a mag-hálón a distributeGraph függvény segítségével.

| Függvény | Cél | Megvalósítási Részlet |
| :--- | :--- | :--- |
| distributeGraph | Csomópontokat oszt el a magok között. | Csomópontokat rendel magokhoz a node_id % core_count alapján. |
| synchronizeCores | Konzisztenciát biztosít a NoC-on keresztül. | Feldolgozza az összes függőben lévő üzenetet a NoC sorban. |
| updateEdgeWeights | Tanulási rátákat alkalmaz az élekre. | DynamicEdgeWeighting-et használ az EdgeQuality beállításához. |

Dinamikus Élsúlyozás
A DynamicEdgeWeighting struktúra online tanulási mechanizmust valósít meg a gráf éleihez. Beállítja egy EdgeQuality objektum strength és confidence értékét egy megadott learning_rate segítségével.

3. Fejlett Feldolgozási Modulok

Gráf Izomorfizmus Processzor
A GraphIsomorphismProcessor felelős a gráftöredékek közötti strukturális hasonlóságok azonosításáért.
- Kanonikus Forma: canonical_form karakterláncot generál részgráfokhoz a szomszéd azonosítók és éltípusok rendezésével, lehetővé téve a strukturális hashek O(1) összehasonlítását.
- Izomorfizmus Ellenőrzés: Az areIsomorphic függvény összehasonlítja két gráf kanonikus reprezentációit.

Teljesítmény- és Aktiváláskezelés
Az energiafogyasztás optimalizálásához az RGPU két vezérlőt alkalmaz:
1. PowerGatingController: Monitorozza a mag kihasználtságát és power_gated állapotba helyezi a magokat, ha azok a power_threshold alá esnek.
2. SparseActivationManager: Nyomon követi a csomópontok aktivitási szintjeit. A sparsity_threshold alatti aktivitású csomópontok deaktiválódnak a feldolgozási ciklusok megtakarítása érdekében.

4. Telemetria és Statisztikák

Az RGPUStatistics struktúra összesíti a teljesítményadatokat az egész hálón. A statisztikákat a collectStatistics metódus gyűjti, amely végigiterál az összes magon:
- total_energy_consumed: Az összes mag által felhasznált energia összege.
- average_load: Átlagos munkaterhelés a hálón.
- message_count: A NoC által feldolgozott üzenetek teljes száma.
- active_cores: Az idle vagy power_gated állapotban nem lévő magok száma.

---

4.3. VEKTOR FELDOLGOZÓ EGYSÉG (VPU) ÉS NUMERIKUS MOTOR

A Vektor Feldolgozó Egység (VPU) a DANCING keretrendszer elsődleges numerikus gyorsítójaként szolgál. Nagy teljesítményű SIMD (Single Instruction, Multiple Data) absztrakciókat, mátrix-dekompozíciós rutinokat és speciális relációs vektorműveleteket biztosít, amelyeket gráf-vektorizáláshoz és spektrális beágyazáshoz használnak.

1. SimdVector Generikus Típus

A VPU magja a SimdVector generikus struktúra, amely hardver-specifikus SIMD utasításokat absztrahálja a Zig @Vector beépített elemével. Több numerikus típust és sávszámot támogat, elsősorban az F32x4, F64x4 és I32x8 konfigurációkat célozva a teljesítménykritikus számításokhoz.

Támogatott Vektortípusok
A VectorType enum meghatározza az elérhető konfigurációkat és metaadatokat biztosít a memória-igazításhoz és méretezéshez.

| Típus | Sávok | Elem Mérete | Igazítás |
| :--- | :--- | :--- | :--- |
| f32x4 | 4 | 4 bájt | 16 bájtos |
| f32x8 | 8 | 4 bájt | 32 bájtos |
| f64x2 | 2 | 8 bájt | 16 bájtos |
| f64x4 | 4 | 8 bájt | 32 bájtos |
| i32x4 | 4 | 4 bájt | 16 bájtos |
| i32x8 | 8 | 4 bájt | 32 bájtos |

Kulcs SimdVector Műveletek
A SimdVector szabványos aritmetikai és fejlett numerikus primitíveket valósít meg:
- Aritmetika: add, sub, mul és divChecked.
- Geometriai: dot szorzat, magnitude és normalize.
- Hardveres Gyorsítás: fma (Fused Multiply-Add) a @mulAdd segítségével.
- Redukciók: @reduce(.Add, product) a dot szorzatokhoz a vízszintes összeadási utasítások kihasználásához.

2. Mátrixműveletek (MatrixOps)

A MatrixOps csomag optimalizált lineáris algebrai rutinokat biztosít, amelyek kifejezetten 4x4-es mátrixokhoz vannak hangolva, amelyeket gyakran használnak kvantumállapot-transzformációkban és gráf-koordináta-rendszerekben.

matmul4x4Simd
Ez a függvény 4x4-es mátrixszorzást végez azáltal, hogy minden sort SimdVector(f32, 4)-ként kezel. A VPU FMA (Fused Multiply-Add) képességeit használja az utasítások számának minimalizálásához és az átviteli sebesség maximalizálásához.

QR Dekompozíció és Inverz
Lineáris rendszerek megoldásához és gráf-beágyazások ortonormalizálásához a MatrixOps tartalmazza:
- QR Dekompozíció: Egy mátrixot ortogonális (Q) és felső háromszögű (R) mátrixra bont.
- Inverz: Kiszámítja a mátrix inverzét, ami elengedhetetlen a ZRuntime-ban lévő transzformációk megfordításához.

3. Relációs Vektorműveletek

A RelationalVectorOps áthidalja a nyers numerikus feldolgozás és a SelfSimilarRelationalGraph közötti szakadékot. Ezeket a műveleteket a gráfstruktúrákból való szemantikai jelentés levezetésére használják.

- Csomópont-hasonlóság: Kiszámítja a VPU-ban tárolt csomópontjellemző-vektorok koszinusz-hasonlóságát vagy Euklideszi távolságát.
- Gráf-vektorizálás: Részgráfokat nagy dimenziós vektorokká alakít a ChaosCoreKernel általi feldolgozáshoz.
- Spektrális Beágyazás: Kiszámítja a gráf Laplace-mátrixának sajátvektorait a gráf alacsony dimenziós térbe való vetítéséhez, megkönnyítve a klaszter-detektálást és a szimmetria-elemzést.

4. Memória- és Gyorsítótárazási Infrastruktúra

A SIMD teljesítmény fenntartásához a VPU szigorú memória-igazítást és hatékony adatlekérést igényel.

MemoryPool
A MemoryPool biztosítja, hogy minden vektoradat 32 bájtos igazítással legyen lefoglalva. Ez megakadályozza a nem igazított SIMD betöltések/tárolások teljesítménybüntetéseit olyan architektúrákon, mint az x86_64 (AVX) és az ARM (NEON).

VectorCache (LRU)
A VectorCache Legkevésbé Nemrég Használt (LRU) kiürítési politikát valósít meg a gyakran hozzáfért vektorokhoz. Ez csökkenti a csomópontbeágyazások vagy átmeneti mátrixok lekérésének késleltetését a nagy frekvenciájú következtetési hurkok során.

5. Logaritmikus Számrendszer (LNS)

A VPU támogatja a Logaritmikus Számrendszert (LNS), amely a számokat logaritmusaikkal reprezentálja. Ez különösen hasznos a SignalPropagationEngine számára, ahol sok kis valószínűség vagy amplitúdó szorzása alulcsorduláshoz vezethet a szabványos lebegőpontos aritmetikában.

- LNSInstruction: Műveleteket foglal magában a log tartományban (pl. az összeadás LNS-ben összetett nemlineáris művelet, míg a szorzás egyszerű összeadássá válik).
- Teljesítmény: Az LNS nagy dinamikatartományú számításokat tesz lehetővé 64 bites lebegőpontok terhelése nélkül.

6. VPU Statisztikák és Telemetria

A VPUStatistics struktúra nyomon követi a numerikus motor működési állapotát és teljesítményét:
- SIMD Kihasználtság: A SIMD utasítások és a skaláris tartalékok aránya.
- Gyorsítótár Találati Arány: A VectorCache hatékonysága.
- Átviteli Sebesség: GFLOPS (Giga Lebegőpontos Műveletek Másodpercenként) mérve mátrixműveletek során.

---

4.4. JELTERJEDÉSI MOTOR

A Jelterjedési Motor (SPE) egy alapvető végrehajtási alrendszer, amely felelős az információáramlás szimulálásáért a SelfSimilarRelationalGraph-on keresztül. A statikus gráfbejárásokkal ellentétben az SPE a csomópontokat oszcilláló entitásokként kezeli komplex állapotokkal, ahol a jelek az élsúlyok, kvantumfázisok és temporális késleltetések alapján terjednek. Egy "gyűjtés-majd-alkalmazás" modellt valósít meg a determinisztikus frissítések biztosítása érdekében diszkrét időlépések során.

Jelállapot és Oszcilláló Modell

A motorban lévő információ a SignalState struktúrában van beágyazva. Minden jelet hullámszerű tulajdonságok jellemeznek, lehetővé téve a konstruktív és destruktív interferencia mintákat a terjedés során.

| Tulajdonság | Típus | Leírás |
| :--- | :--- | :--- |
| amplitude | f64 | A jel nagysága, amely befolyásolja a qubit gerjesztést. |
| phase | f64 | A szögi komponens (0-tól 2π-ig) az interferenciához. |
| frequency | f64 | A fázisváltozás sebessége az idő függvényében. |
| timestamp | i128 | Nagy felbontású nanoszekundumos időbélyeg a jel létrehozásának/frissítésének időpontjáról. |

A jelek az advance(delta_time) segítségével fejleszthetők az idő múlásával, amely frissíti a fázist a frekvencia alapján. Egyetlen csomóponthoz érkező több jel a combine() segítségével egyesül, amely átlagolja fizikai tulajdonságaikat az eredő állapot meghatározásához.

---

Motor Életciklus és Vezérlés

A SignalPropagationEngine kezeli a terjedés állapotát a gráfon keresztül. activation_traces-t tart fenn a jelmozdulás előzményeinek nyomon követéséhez és integrálódik a DataFlowAnalyzer-rel a memória-hozzáférési minták optimalizálásához.

Kulcs Életciklus Függvények:
- init: Inicializálja a motort a gráfra és a DataFlowAnalyzer-re mutató hivatkozással.
- initiateSignal: Kezdeti SignalState-et injektál egy specifikus forráscsomópontba, beállítja a kezdeti fázist és rögzíti az első aktiválási nyomot.
- propagateStep: A szimuláció egyetlen diszkrét időlépését hajtja végre. Kiszámítja, hogyan mozognak a jelek az aktív csomópontokból a szomszédaikba az élsúlyok és a propagation_speed alapján.
- propagateMultipleSteps: Egy burkoló, amely ciklusban hajtja végre a propagateStep-et, minden iterációban time_step-pel növelve a current_time-ot.

Terjedési Logika Folyamata
A motor egy kétfázisú "gyűjtés-majd-alkalmazás" modellt használ a versenyhelyzetek megelőzésére és annak biztosítására, hogy egyetlen időlépés összes csomópontja ugyanarra a globális állapotra reagáljon.

1. Gyűjtési Fázis: Azonosítja az összes jelenleg "aktív" csomópontot (azokat, amelyeknek közelmúltbeli jelei vannak) és kiszámítja az összes kapcsolódó szomszéd kimenő jeleit.
2. Alkalmazási Fázis: Egyszerre frissíti a célcsomópontok SignalState, qubit amplitúdó és phase értékeit.

---

Elemzés és Integráció

Aktiválási Nyomkövetés
Az ActivationTrace struktúra temporális naplóként működik minden csomóponthoz. Tárolja a signal_history-t (SignalState-ek ArrayList-je) és metrikákat számít, mint például:
- Átlagos Amplitúdó/Frekvencia: Hasznos a gráfban lévő rezonancia azonosításához.
- Időtartam: Az első és utolsó aktiválás közötti időszak egy csomópontnál.

Terjedési Statisztikák
A PropagationStatistics struktúra nyomon követi a motor globális hatékonyságát. Monitorozza a unique_nodes_activated és average_propagation_speed értékeket. Ezek a metrikák kritikusak a ReasoningOrchestrator számára annak meghatározásához, hogy a gráf elért-e stabil állapotot, vagy szükséges-e további optimalizálás.

Integráció a ChaosCore-ral
A motor integrálódik a DataFlowAnalyzer-rel. Minden alkalommal, amikor egy jel aktivál egy csomópontot, a motor értesítheti az elemzőt, lehetővé téve a ChaosCoreKernel számára, hogy optimalizálja a nagy aktivitású részgráfok (hotspotok) fizikai memóriaelhelyezését a késleltetés csökkentése érdekében a következő terjedési lépésekben.

---

5.1. CREV CSŐVEZETÉK: TUDÁSKINYERÉS ÉS INDEXELÉS

A CREV (Relációs Kinyerés és Ellenőrzés) Csővezeték a DANCING keretrendszer elsődleges adatbetöltési motorja. Strukturálatlan vagy félig strukturált bemenetet alakít át nagy hűségű RelationalTriplet struktúrákká, amelyeket ezt követően integrálnak a SelfSimilarRelationalGraph-ba. A csővezeték egy többlépéses életciklust alkalmaz, amely magában foglalja a morfémaalapú szótövesítést, a Welford-algoritmus segítségével végzett statisztikai anomáliadetektálást és a konfidencia-súlyozott integráción keresztüli konfliktusfeloldást.

RelationalTriplet és Adatstruktúrák

A CREV csővezetékben a tudás alapvető egysége a RelationalTriplet. A szabványos RDF hármasokkal ellentétben ezek temporális metaadatokat és konfidencia-pontszámot tartalmaznak, amely közvetlenül befolyásolja az eredményül kapott gráfélek kvantumállapotát.

| Típus | Leírás |
| :--- | :--- |
| RelationalTriplet | Egy struktúra, amely subject, relation, object (karakterláncként), confidence (f64) és extraction_time (i128) értékeket tartalmaz. |
| KnowledgeGraphIndex | Egy speciális hármas tároló, amely a hármas identitás hasheket teljes metaadataikra képezi le a gyors kereséshez. |
| StreamBuffer | Egy körkörös puffer, amelyet a bejövő adatfolyamok csúszóablakos elemzéséhez használnak. |

Kinyerési Életciklus

A csővezeték öt különböző szakaszon dolgozza fel az adatokat, amelyeket az ExtractionStage enum határoz meg. Minden szakasz érvényesítésre kerül, mielőtt az adatok a következő fázisba lépnek.

1. Tokenizálás és Szótövesítés
A csővezeték WordToken struktúrákra bontja a nyers szöveget. Egyedi, morfémaalapú szótövesítési algoritmust (stemWord) alkalmaz, amely kezeli a közönséges angol utótagokat (pl. "-ting", "-ing", "-ated", "-ment") a tokenek normalizálásához a mintaillesztés előtt.

2. Hármas Kinyerés
A mintaillesztés a matchPatternMorphemeAware segítségével történik, amely összehasonlítja a mondattokeneket az ismert relációs mintákkal a normalizált szótövek segítségével.

3. Érvényesítés és Anomáliadetektálás
A kinyert hármasok statisztikai érvényesítőn mennek keresztül. A csővezeték Welford-algoritmust alkalmaz a hármas konfidenciaszintek futó statisztikáinak (átlag és variancia) fenntartásához az egész előzmény tárolása nélkül. Az anomáliadetektálás egy új hármas konfidenciájának Z-pontszámának kiszámításával történik; ha a pontszám meghalad egy előre meghatározott küszöbértéket, a hármas manuális felülvizsgálatra vagy alacsonyabb súlyú integrációra kerül megjelölésre.

4. Integráció és Konfliktusfeloldás
Amikor egy új hármas ütközik a meglévő tudással (azonos Alany-Reláció, de eltérő Tárgy), a resolveConflict függvény kerül meghívásra. Összehasonlítja a confidence pontszámokat és az extraction_time értékeket. A magasabb konfidenciájú hármasok általában felülírják a régebbi, alacsonyabb konfidenciájú adatokat, bár a ChaosCoreKernel dönthet úgy, hogy mindkettőt megőrzi összefonódott állapotban, ha a meglepetési metrika magas.

5. Indexelés
A véglegesített hármasok a KnowledgeGraphIndex-ben tárolódnak. Ez az index kétféle hasht generál:
- Identitás Hash: Csak az A-R-T alapján, deduplikáláshoz.
- Mező Hash: Tartalmazza a konfidenciát és az időbélyeget, verziókövetéshez.

---

Rendszerintegráció

ChaosCoreKernel Memóriafoglalás
A CREVPipeline nem foglal memóriát közvetlenül az operációs rendszertől. Ehelyett memóriablokkokat kér a ChaosCoreKernel-től. Ez biztosítja, hogy minden tudáskinyerési tevékenységet nyomon követ a DataFlowAnalyzer és a SurpriseMemoryManager kiürítési politikáinak hatálya alá esik.

InferenceHook Visszahívások
A csővezeték megvalósítja az InferenceHook interfészt. Ez lehetővé teszi más alrendszerek (pl. a SignalPropagationEngine) számára, hogy feliratkozzanak a kinyerési eseményekre. Amikor egy hármas sikeresen integrálódik, a csővezeték aktiválja az onExtractionComplete-et, amely jelimpulzust indíthat a gráfon a kapcsolódó csomópontfázisok frissítéséhez.

Kulcsfüggvények

hashTripletFields
SHA-256 hasht generál a hármas tartalmából, beleértve a konfidenciát és a kinyerési időt. Ez a tudásbázis integritásának biztosítására szolgál.

stemWord
Egy szabályalapú szótövesítő, amely kezeli a technikai relációs kinyeréshez szükséges specifikus nyelvi mintákat. Csökkenti a matchPatternMorphemeAware függvény összetettségét azáltal, hogy kanonikus formát biztosít az igékhez és főnevekhez.

Welford-algoritmus Megvalósítása
Az érvényesítési szakaszban az átlag (μ) és a variancia (σ²) inkrementális kiszámítására használják:
1. M_n = M_(n-1) + (x_n - M_(n-1)) / n
2. S_n = S_(n-1) + (x_n - M_(n-1))(x_n - M_n)

---

5.2. ZRUNTIME: VÁLTOZÓ ÉS RELÁCIÓS LOGIKAI KÖRNYEZET

A ZRuntime a DANCING keretrendszer magas szintű végrehajtási környezeteként és állapotkoordinátoraként szolgál. Kezeli a ZVariable-ok életciklusát, amelyek hibrid adatstruktúrák, amelyek egy relációs gráfot, kvantumlogikai állapotot és temporális előzmény-auditnaplót foglalnak magukban. A ZRuntime mechanizmusokat biztosít a relációs műveletekhez (AND/OR/XOR), az információ terjedéséhez a változóhatárokon keresztül és az összetett relációs kifejezések végrehajtásához.

ZVariable Életciklus és Komponensek

A ZVariable a ZRuntime elsődleges állapotegysége. Minden változó fenntartja saját helyi topológiáját és kvantumállapotát, lehetővé téve az izolált manipulációt, mielőtt nagyobb relációs struktúrákba integrálódna.

- SelfSimilarRelationalGraph: Minden változó dedikált gráfpéldányt tartalmaz a belső relációs függőségek megjelenítéséhez.
- RelationalQuantumLogic: Kezeli a változóra specifikus kvantumállapotot és kapu-műveleteket.
- History: HistoryEntry objektumok lineáris auditnapló-ja, amely nyomon követi az assign, transform, relate, measure és entangle eseményeket.

Változó Életciklus Folyamata:
1. Inicializálás: A ZVariable.init lefoglalja a gráf- és logikai komponenseket.
2. Hozzárendelés: Az assign műveletek rögzítik az állapotváltozásokat az előzménynaplóban.
3. Reláció: A relateTo éleket hoz létre a változók között, potenciálisan aktiválva az információterjedést.
4. Mérés: A measure összeomlasztja a belső kvantumállapotot klasszikus reprezentációvá.

Relációs Logika és Műveletek

A ZRuntime egyedi logikai rendszert valósít meg, ahol a klasszikus boolean műveletek kiterjesztve működnek gráftopológiákon és kvantumállapotokon.

Relációs Műveleti Mechanika
A relationalOperation függvény logikát hajt végre két ZVariable példány között egy operation_type (AND, OR, XOR, ENTANGLE) alapján.

| Művelet | Leírás |
| :--- | :--- |
| AND | Metszi két változó relációs gráfjait, megtartva a közös csomópontokat és éleket. |
| OR | Egyesíti a relációs gráfokat, létrehozva a topológiák unióját. |
| XOR | Megtartja az egyik vagy másik változóra egyedi csomópontokat és éleket. |
| ENTANGLE | Kvantum-összefonódási kapcsolatot hoz létre a változók RelationalQuantumLogic állapotai között. |

Információterjedés
Az információ a propagate_information segítségével terjed a változóhatárokon keresztül. Ez a mechanizmus biztosítja, hogy az egyik változó fraktáldimenziójának vagy élsúlyainak változásai befolyásolják a kapcsolódó szomszédait a ZRuntime környezetben.

---

Relációs Kifejezés Elemzése és Végrehajtása

A ZRuntime tartalmaz egy elemzőt és végrehajtót a relációs kifejezésekhez, lehetővé téve a felhasználók számára, hogy összetett gráftranszformációkat definiáljanak egy tartomány-specifikus logikán keresztül.

- computeRelationalExpression: Ez a függvény elemzi a relációs logikát képviselő karakterláncokat (pl. (A AND B) OR C) és feloldja azokat az érintett ZVariable példányokon végzett szekvenciális műveletek végrehajtásával.
- executeQuantumCircuit: Közvetlenül interfészeli egy változó RelationalQuantumLogic-ját LogicGate műveletek kötegének alkalmazásához (Hadamard, Pauli-X, CNOT stb.).
- Fraktáldimenzió Metrikák: A végrehajtás során a futtatókörnyezet monitorozza az alapul szolgáló gráfok fraktáldimenzióját a strukturális integritás és összetettségi korlátok biztosítása érdekében.

Végrehajtási Audit és Előzmények

A hibrid számítási környezet átláthatóságának fenntartásához minden műveletet strukturált auditnapló rögzít.

ExecutionHistoryEntry Struktúra
Az ExecutionHistoryEntry rögzíti egy futásidejű művelet teljes kontextusát:
- Művelettípus: Az ExecutionAction enum határozza meg (pl. create_variable, fractal_transform).
- Célok: Azonosítja az elsődleges változót és a műveletben érintett másodlagos változókat.
- Eredmények: Az eredményt karakterláncként, egészként vagy lebegőpontosként tárolja a következő elemzéshez vagy hibakereséshez.
- Időbélyeg: Nanoszekundumos pontosság a std.time.nanoTimestamp() segítségével.

Technikai Adatfolyam Összefoglalása

| Fázis | Komponens | Logika |
| :--- | :--- | :--- |
| Elemzés | computeRelationalExpression | Tokenizálja a relációs karakterláncokat ExecutionAction sorozattá. |
| Végrehajtás | relationalOperation | Diszpécsel a SelfSimilarRelationalGraph-hoz a topológiához vagy a RelationalQuantumLogic-hoz az összefonódáshoz. |
| Terjedés | propagate_information | Frissíti az élminőségeket és csomópontfázisokat a változó-függőségi gráfon keresztül. |
| Naplózás | ExecutionHistoryEntry | Rögzíti a transzformációs deltát és tárolja a változó history-jában. |

---

5.3. MEGLEPETÉS MEMÓRIAKEZELŐ

A Meglepetés Memóriakezelő, amelyet a surprise_memory.zig valósít meg, egy kognitív ihletésű memória-alrendszer, amely az adatmegőrzést az információs "meglepetés" alapján prioritizálja, nem pedig egyszerű recency vagy frekvencia alapján. Integrálódik a Tartalom-Címezhető Tárolással (CAS) és a DataFlowAnalyzer-rel a memóriablokkok kezeléséhez azok meglévő tudástól való eltérésének kiszámításával, biztosítva, hogy az újszerű vagy anomális adatok megőrződjenek, míg a redundáns információk kiürítésre kerülnek.

1. Meglepetési Metrikák és Különbözőség-számítás

A rendszer három elsődleges dimenzió segítségével értékeli egy memóriablock "meglepetését", amelyeket a SurpriseMetrics struktúra foglal magában.

| Metrika | Megvalósítási Részlet | Leírás |
| :--- | :--- | :--- |
| Jaccard Különbözőség | jaccard_dissimilarity | Méri a tartalomjellemzők (pl. bigramok) metszet-az-unión arányát egy alapvonalhoz képest. |
| Tartalomhash Távolság | content_hash_distance | Normalizált bit-távolság a blokk SHA-256 hashje és a meglévő klaszterközéppontok között. |
| Temporális Újdonság | temporal_novelty | Bomlásalapú pontszám az alapján, hogy mennyi idő telt el azóta, hogy hasonló mintát utoljára megfigyeltek. |

A combined_surprise pontszám ennek a három befogott egységértéknek a számtani átlaga.

2. Megőrzési Prioritás és Életciklus

Minden memóriablokkot egy SurpriseRecord követ nyomon, amely fenntartja életciklusát és kiszámít egy dinamikus retention_priority értéket.

Megőrzési Képlet
A megőrzési prioritás nem statikus; minden hozzáférésnél újraszámítják a következő súlyozott komponensek segítségével:
- Alapsúly (0.5): A belső meglepetési pontszám.
- Recency Tényező (0.3): 1.0 / (1.0 + age_ms) képlettel számítva.
- Frekvencia Tényező (0.2): freq / (freq + 8.0) képlettel számítva, telítettséget biztosítva a nagy frekvenciájú hozzáféréshez.

3. Kiürítési és Szervezési Stratégiák

A kezelő speciális algoritmusokat alkalmaz a memória egészségének fenntartásához, amikor a korlátok elérésre kerülnek.

Részleges Rendezéses Kiürítés
Az evictLowSurpriseBlocks függvény részleges rendezési stratégiát alkalmaz a legalacsonyabb retention_priority értékű blokkok azonosításához. A szabványos LRU-val ellentétben ez biztosítja, hogy egy nagyon "meglepő" blokk (pl. egy ritka anomália) megőrződjön, még ha nemrég nem is fértek hozzá, míg az "unalmas", de gyakori adatok prioritást kapnak a kiürítéshez.

Összefonódás-alapú Szervezés
Az organizeByEntanglement függvény azonosítja a magas meglepetési értékű blokkok párjait, amelyek gyakran együtt jelennek meg a DataFlowAnalyzer nyomokban.
- Párosítás: A magas kölcsönös információval vagy temporális közelséggel rendelkező blokkok összekapcsolódnak.
- Klaszterezés: Ezek a párok prioritást kapnak a ContentAddressableStorage-ban való együttes elhelyezéshez, minimalizálva a lekérési késleltetést összetett következtetési feladatok során.

4. Rendszerintegráció és Szálbiztonság

A SurpriseMemoryManager közvetítőként szolgál a CREVPipeline és a ChaosCoreKernel között.

| Komponens | Interakció Típusa | Cél |
| :--- | :--- | :--- |
| ContentAddressableStorage | Tárolási Backend | Biztosítja a nyers MemoryBlock tárolást és hash-alapú indexelést. |
| DataFlowAnalyzer | Mintabemenet | Hozzáférési mintákat biztosít a temporális újdonság és az összefonódás kiszámításához. |
| Mutex | Szinkronizálás | Szálbiztonságot biztosít a SurpriseRecord frissítések és kiürítési ciklusok során. |

5. Technikai Korlátok

- Maximális Bemeneti Méret: 100 MB.
- Jaccard Mintaméret: 1000 jellemző.
- Temporális Ablak: 86 400 másodperc (24 óra) az újdonság bomlásához.
- Pontosság: f64-et használ minden prioritásszámításhoz, 1e-12 FLOAT_EPSILON értékkel a stabilitáshoz.

---

6. KVANTUMSZÁMÍTÁSI RÉTEG

A Kvantumszámítási Réteg hibrid kvantum-relációs végrehajtási képességekkel látja el a DANCING keretrendszert. Ez a réteg áthidalja a magas szintű gráfstruktúrákat és a relációs logikát mind a helyi kvantumszimulációval, mind a fizikai kvantumhardverrel. Lehetővé teszi a rendszer számára a relációs műveletek állapotvektor-szimulációjának elvégzését és az összetett optimalizálási feladatok kiszervezését IBM Quantum hardverre.

Rendszer Összekapcsolhatóság

A kvantumréteg speciális társprocesszorként működik a SelfSimilarRelationalGraph számára. Azonosítja a magas összefonódással vagy fraktálkomplexitással rendelkező részgráfokat, és kvantumáramkörökké alakítja azokat optimalizálás céljából.

Támogatott Kvantumlogikai Kapuk

| Kaputípus | Qubitek | Leírás |
| :--- | :--- | :--- |
| HADAMARD | 1 | Szuperpozíciót hoz létre. |
| CNOT | 2 | Feltételes NOT (Összefonódás). |
| RELATIONAL_AND | 2 | Kvantum relációs metszet. |
| FRACTAL_TRANSFORM | 1 | Önhasonlóság-skálázást alkalmaz. |

IBM Backend Specifikációk (Átlagértékek)

| Backend Család | T1 (ns) | T2 (ns) | Leolvasási Hiba |
| :--- | :--- | :--- | :--- |
| HERON | 350 000 | 200 000 | 0,008 |
| EAGLE | 200 000 | 120 000 | 0,015 |
| FALCON | 100 000 | 80 000 | 0,020 |

---

6.1. KVANTUMLOGIKA SZIMULÁCIÓS MOTOR

A Kvantumlogika Szimulációs Motor a DANCING architektúrán belüli kvantumállapotok és műveletek szimulálásának alapvető numerikus és logikai keretrendszere. Matematikai alapot biztosít a qubit-reprezentációhoz, a komplex amplitúdó-manipulációhoz és a relációs logikai kapuk speciális készletéhez, amelyek áthidalják a klasszikus logikát a kvantum szuperpozícióval.

Kvantumállapot Reprezentáció

A QuantumState struktúra a szimuláció alapvető egysége, amely egyetlen qubitet vagy egy összefonódott rendszer komponensét képviseli. A szabványos szimulátorokat meghaladóan, amelyek csak állapotvektorokat követnek nyomon, ez a motor explicit módon nyomon követi a fázist és az összefonódási metrikákat a relációs gráf optimalizálásának megkönnyítéséhez.

QuantumState Adatstruktúra

| Mező | Típus | Leírás |
| :--- | :--- | :--- |
| amplitudes | [2]Complex(f64) | Az α és β valószínűségi amplitúdók, ahol |α|² + |β|² = 1. |
| phase | f64 | A kvantumállapot relatív fázisa. |
| entanglement_degree | f64 | Skaláris érték, amely más állapotokkal való korreláció erősségét képviseli. |

Kulcsműveletek:
- Normalizálás: Biztosítja, hogy a teljes valószínűségi nagyság egyenlő 1,0-val a normalize() segítségével.
- Bázis Inicializálás: Az initBasis(is_one, ...) tiszta |0> vagy |1> állapotokat hoz létre.
- Állapot Matematika: Támogatja az add, scale és conjugate műveleteket a vektortér-manipulációkhoz.

Relációs Kvantumlogika (RQL)

A RelationalQuantumLogic struktúra kezeli a kvantumállapotok életciklusát és az áramkörök végrehajtását. Ez az elsődleges interfész a kapuk alkalmazásához és a mérések elvégzéséhez.

LogicGate Enum
A motor szabványos kvantumkapukat támogat, egyedi "Relációs" és "Fraktál" változatokkal együtt, amelyeket gráf-alapú logikához terveztek.

| Kategória | Kapuk |
| :--- | :--- |
| Szabványos Egyqubites | HADAMARD, PAULI_X, PAULI_Y, PAULI_Z, PHASE |
| Szabványos Többqubites | CNOT, TOFFOLI |
| Relációs | RELATIONAL_AND, RELATIONAL_OR, RELATIONAL_XOR, RELATIONAL_NOT |
| Strukturális | FRACTAL_TRANSFORM |

Végrehajtás és Szerializálás
A motor kötegelt feldolgozást támogat a QuantumCircuit segítségével és perzisztenciát bináris szerializálási formátumon keresztül.

1. QuantumCircuit: Több LogicGate műveletet kötegelve atomikus végrehajtáshoz.
2. Koherencia Küszöbérték: Biztonsági mechanizmus, amely monitorozza az entanglement_degree és phase értékeket a szimulációs stabilitás biztosítása érdekében.
3. Bináris Szerializálás: A serialize és deserialize függvények lehetővé teszik a RelationalQuantumLogic állapot tárolását a ContentAddressableStorage-ban (CAS) vagy az AsynchronousNoC-on keresztüli átvitelét.

Technikai Specifikációk

Koherencia és Összefonódás
A motor fenntart egy coherence_threshold értéket. Ha egy QuantumState entanglement_degree értéke meghalad bizonyos korlátokat megfelelő normalizálás nélkül, a RelationalQuantumLogic rendszer újranormalizálási menetet indít a numerikus divergencia megelőzése érdekében.

Kapu Auditnaplója
Minden applyGate által végrehajtott műveletet hozzáfűznek a GateHistoryEntry listához. Ez a napló tartalmazza:
- A LogicGate típusát.
- A célqubit indexeket.
- Egy időbélyeget vagy szekvencia azonosítót.
- A coherence metrika állapotát a művelet után.

---

6.2. KVANTUMHARDVER ABSZTRAKCIÓ ÉS IBM INTEGRÁCIÓ

A Kvantumhardver Absztrakciós réteg egységes interfészt biztosít a kvantumáramkörök végrehajtásához mind helyi szimulátorokban, mind fizikai IBM Quantum hardveren. Kezeli az áramkörök OpenQASM 3.0-ba való szerializálását, az IBM Cloud-hoz való hitelesítést és feladatbeküldést, valamint kifinomult eszközöket biztosít a hardver hűségbecsléshez és a hibrid kvantum-klasszikus optimalizáláshoz.

Kvantum Backend Specifikáció

A rendszer a QuantumBackendType enum segítségével kategorizálja a hardvert, amely specifikus IBM architektúrákat és szimulátorokat tartalmaz. Minden hardvertípushoz részletes kalibrációs metrikák tartoznak, beleértve a T1 és T2 relaxációs időket, leolvasási hibaarányokat és ECR kapu hibákat.

| Backend Típus | Qubit Szám | T1 Átlag (ns) | T2 Átlag (ns) | Leolvasási Hiba |
| :--- | :--- | :--- | :--- | :--- |
| HARDWARE_HERON | 133 | 350 000 | 200 000 | 0,008 |
| HARDWARE_EAGLE | 127 | 200 000 | 120 000 | 0,015 |
| HARDWARE_FALCON | 27 | 100 000 | 80 000 | 0,020 |
| SIMULATOR | 32 | végtelen | végtelen | 0,000 |

IBM Quantum Integráció

Az IBMQuantumClient kezeli a távoli végrehajtás életciklusát, a hitelesítő adatok kezelésétől a feladateredmény lekéréséig. Egy std.http.Client-et használ az IBM Quantum API-val való kommunikációhoz.

Adatfolyam: Feladatbeküldés
1. Hitelesítés: Az ügyfél Bearer tokent és Cloud Resource Name (CRN) azonosítót használ az identitáskezeléshez.
2. Szerializálás: Az áramkörök OpenQASM 3.0-ra konvertálódnak. Az ügyfél JSON escape segédprogramot tartalmaz a QASM karakterláncok biztonságos beágyazásához a HTTP hasznos adatba.
3. Beküldés: A submitJobWithBackend POST kérést küld az IBM Cloud feladatok végpontjára a QASM hasznos adattal és a lövésszámmal.
4. Lekérdezés: A rendszer a getJobResult segítségével kéri le az eredményeket a beküldés során visszaadott feladatazonosító alapján.

Biztonság és Hitelesítő Adatok Törlése
A hitelesítő adatok kiszivárgásának megelőzése érdekében az IBMQuantumClient megvalósítja a zeroSensitiveData függvényt, amely @memset-et használ az api_token és crn pufferek felülírásához a memóriában a felszabadítás előtt.

Hardver Hűség és Szimuláció

Az absztrakciós réteg tartalmaz egy StateVectorSimulator-t a helyi végrehajtáshoz és egy FidelityEstimator-t a valós hardveren való teljesítmény előrejelzéséhez.

Megvalósítási Entitások:
- StateVectorSimulator: 2^n méretű komplex amplitúdóvektort kezel. Támogatja a kapu-alkalmazásokat és a mérési valószínűségeket.
- FidelityEstimator: Kiszámítja egy áramkör várható sikervalószínűségét az IBMBackendCalibrationData alapján. Figyelembe veszi a mélységfüggő dekoherenciát és a kapu hiba felhalmozódást.

Hibrid Optimalizálás (VQE/QAOA)

A QuantumClassicalHybridOptimizer megvalósítja a Paraméter-Eltolás Szabályt a Variációs Kvantum Sajátérték-megoldó (VQE) és a Kvantum Közelítő Optimalizálási Algoritmus (QAOA) munkaterhelések gradienseinek kiszámításához.

Kulcskomponensek:
- Paraméterkezelés: Nyomon követi a parameters-t f64 szeletként és frissíti azokat egy klasszikus optimalizáló segítségével (pl. gradiens ereszkedés).
- Paraméter-Eltolás Szabály: Egy G(θ) = e^(-iθP/2) kapuhoz a gradiens kiszámítása: ∂E/∂θ = (E(θ + π/2) - E(θ - π/2)) / 2. Ez két áramkörváltozat végrehajtásával valósul meg minden paraméterhez.

Áramkör Felépítés és Szerializálás

A QuantumCircuit struktúra biztosítja az elsődleges interfészt a kvantumprogramok felépítéséhez. QuantumGate utasítások listáját tartja fenn és egy specifikus QuantumBackendType-ra képezi le azokat.

Szerializálási Logika - A serializeToOpenQASM3 függvény a belső kapu listát szabványos karakterlánc formátumra konvertálja:

| Kaputípus | QASM 3.0 Kimenet |
| :--- | :--- |
| HADAMARD | h q[i]; |
| CNOT | cx q[i], q[j]; |
| PHASE | p(theta) q[i]; |
| MEASURE | c[i] = measure q[i]; |

---

6.3. KVANTUMFELADAT ADAPTER

A Kvantumfeladat Adapter hídként szolgál a magas szintű relációs gráfstruktúrák és az alacsony szintű kvantum végrehajtási környezetek között. Felelős a SelfSimilarRelationalGraph azon részeinek azonosításáért, amelyek magas kvantumpotenciált mutatnak (összefonódás és fraktálkomplexitás), kvantumáramkörökké alakítja azokat, és visszaszinkronizálja az eredményeket a gráf állapotába.

Alapvető Adatstruktúrák

QuantumSubgraph
A QuantumSubgraph kvantumoptimalizáláshoz azonosított csomópontok és élek klaszterét képviseli. Összesíti a metrikákat, mint a teljes összefonódás és az átlagos fraktáldimenzió, a hardver kiszervezés alkalmasságának meghatározásához.

| Mező | Típus | Leírás |
| :--- | :--- | :--- |
| node_ids | ArrayList([]const u8) | A klaszteren belüli csomópontok egyedi azonosítói. |
| edge_keys | ArrayList(nsir.EdgeKey) | A csomópontokat összekötő élek kulcsai. |
| total_entanglement | f64 | A részgráfban lévő kvantumkorrelációk kumulatív nagysága. |
| avg_fractal_dimension | f64 | Átlagos fraktáldimenzió a részgráf élein keresztül. |
| semantic_cluster_id | u64 | Élminőségek szemantikai klaszterezéséből levezetett azonosító. |

QuantumTaskAdapter
Az elsődleges orchestrátor, amely kezeli a kvantumfeladatok életciklusát az azonosítástól az eredmény alkalmazásáig.

| Komponens | Szerep |
| :--- | :--- |
| local_simulator | A RelationalQuantumLogic egy példánya szoftver-alapú tartalékhoz. |
| quantum_client | Opcionális mutató az IBMQuantumClient-re hardveres végrehajtáshoz. |
| statistics | AdapterStatistics a telemetria nyomon követéséhez (késleltetés, qubit használat, sikerességi arányok). |
| thresholds | Konfigurálható entanglement_threshold és fractal_threshold a részgráf kiválasztáshoz. |

Részgráf Azonosítás és Szemantikai Klaszterezés

Az adapter szemantikai klaszterezési algoritmust használ a gráfkomponensek QuantumSubgraph objektumokba csoportosításához. Az éleket "vödrökbe" kategorizálja minőségük, fraktáldimenziójuk és korrelációs nagyságuk alapján.

1. Kulcsgenerálás: A semanticClusterKey függvény karakterlánc kulcsot generál (pl. "q1_f2_c3"), amely egy él szemantikai tulajdonságait képviseli.
2. Klaszterezés: A hasonló szemantikai aláírásokkal rendelkező élekkel összekötött csomópontok csoportosítódnak.
3. Metrika Érvényesítés: Egy részgráf isQuantumSuitable-ként van megjelölve, ha total_entanglement értéke meghaladja az entanglement_threshold-ot (alapértelmezett 0,5) és avg_fractal_dimension értéke nagyobb mint 1,5.

Áramkör Generálás (QASM 2.0)

Az adapter OpenQASM 2.0 kompatibilis áramköröket generál egy részgráf relációs állapotának megjelenítéséhez. Az átalakítás egy specifikus mintát követ:
- Hadamard Inicializálás: Minden csomópontot képviselő qubit Hadamard kapuval inicializálódik (h q[i];) az állapotok szuperpozíciójának létrehozásához.
- CNOT Összefonódási Lánc: A részgráf élei CX (CNOT) kapukra képezhetők le. Ez összefonódási láncot hoz létre, amely tükrözi a gráf relációs topológiáját.
- Mérés: Az áramkör mérési műveletekkel zárul az állapot klasszikus regiszterekbe való összeomlasztásához.

Végrehajtási Csővezeték

A runFullQuantumOptimization függvény magas szintű csővezetéket biztosít a gráf optimalizálásához.

1. Azonosítás: Megvizsgálja a gráfot alkalmas részgráfok után.
2. Kiszervezés: Minden részgráfhoz az adapter dönt az executeLocalSimulation és az IBM hardveres kiszervezés között a use_real_backend jelző alapján.
3. Végrehajtás:
   - Helyi: RelationalQuantumLogic-ot használ a kapu műveletek szimulálásához.
   - Távoli: IBMQuantumClient-et használ OpenQASM áramkörök beküldéséhez.
4. Alkalmazás: Az eredmények feldolgozása az applyResultsToGraph segítségével.

Eredmény Alkalmazás
Amikor az eredmények QuantumTaskResult formájában visszaérkeznek, az adapter a következőket frissíti:
- Kvantumállapot: A csomópont qubitjei új komplex amplitúdókkal frissülnek.
- Koherencia: A csomópont koherencia értékei a mérési sikeresség alapján módosulnak.
- Kvantumkorreláció: Az él nagyságok frissülnek az áramkörben mért összefonódás tükrözéséhez.

Telemetria és Statisztikák
Az AdapterStatistics struktúra nyomon követi a kvantumréteg teljesítményét és kihasználtságát:
- total_tasks_submitted: Minden alkalommal növekszik, amikor egy részgráf feldolgozásra kerül.
- avg_execution_time_ms: A feladat késleltetések gördülő átlagaként számítva.
- total_qubits_used: A kvantumfeladatokban érintett csomópontok kumulatív száma.

---

7. ELLENŐRZÉSI ÉS BIZTONSÁGI VEREM

Az Ellenőrzési és Biztonsági Verem átfogó, többrétegű keretrendszert biztosít a hibrid kvantum-relációs gráf matematikai helyességének, strukturális integritásának és kriptográfiai biztonságának biztosítására. Az alacsony szintű formális logikától és típuselméletektől a magas szintű nulla-tudás bizonyítékokig és homomorf titkosításig terjed.

Rendszer Áttekintés

A verem úgy van tervezve, hogy érvényesítse a SelfSimilarRelationalGraph-on belüli minden átmenetet. Biztosítja, hogy a kvantumállapotok normalizáltak maradjanak, a gráftopológiák megfeleljenek az invariánsoknak, és az információáramlás tiszteletben tartsa a biztonsági rácsokat.

7.1 Formális Ellenőrzési Motor
A FormalVerificationEngine felelős annak bizonyításáért, hogy a gráfállapot kielégíti az előre meghatározott InvariantType predikátumok készletét. Egy TheoremProver-t alkalmaz, amely támogatja az egyesítést és a rezolúciós cáfolatot az összetett gráftulajdonságok, mint a kapcsolódás, a kvantumállapot normalizálás és a temporális konzisztencia érvényesítéséhez.
Kulcskomponensek: HoareLogicVerifier a kódútvonal helyességéhez és InvariantRegistry a tulajdonság nyomon követéséhez.

7.2 Típuselmélet és Lineáris Erőforrás Ellenőrzés
Ez a réteg kifinomult típusrendszert valósít meg, amely függő típusokon (Pi és Sigma típusok) és a Curry-Howard megfeleltetésen alapul. A LinearTypeChecker biztosítja, hogy a kvantum erőforrások vagy érzékeny adattokenek specifikus LinearityMode korlátok szerint legyenek felhasználva (pl. LINEAR, AFFINE), megakadályozva az erőforrások illegális duplikálását vagy elvesztését.
Kulcskomponensek: TypeTheoryEngine, LinearTypeChecker és kategóriaelmélet törvény ellenőrzés (Funktor/Monad törvények).

7.3 Biztonsági Bizonyítékok és Információáramlás Elemzés
A SecurityProofEngine kötelező hozzáférés-vezérlést (MAC) és nem-interferencia politikákat érvényesít. Rács-alapú modellt (SecurityLevel és IntegrityLevel) alkalmaz a "Felfelé Olvasás" vagy "Lefelé Írás" sértések (Bell-LaPadula) megelőzésére és az adatintegritás biztosítására a Biba modell segítségével.
Kulcskomponensek: InformationFlowAnalysis, NonInterferenceProver és AccessControlVerifier.

7.4 Nulla-Tudás Ellenőrzés és Ellenőrzött Következtetés
Ez a modul biztosítja a "Kötelezettségvállalás-majd-Bizonyítás" csővezetéket a gráf következtetésekhez. A ZKInferenceProver Groth16 bizonyítékokat generál Circom és snarkjs segítségével, lehetővé téve a rendszer számára annak ellenőrzését, hogy egy specifikus következtetés helyesen lett kiszámítva anélkül, hogy felfedné az alapul szolgáló érzékeny súlyokat vagy bemeneti adatokat.
Kulcskomponensek: ZKProofBundle, CircomProver és VerifiedInferenceEngine.

7.5 Adatkészlet Elhomályosítás és Adatvédelmi Keretrendszer
Az adatvédelmi réteg eszközöket biztosít a titkosítva vagy névtelenül maradó adatok feldolgozásához. Megvalósítja a Paillier kriptoszisztémát a HomomorphicEncryption-hoz, lehetővé téve az additív műveleteket a titkosított szövegen. Kezeli a DatasetFingerprint generálást Lokalitás-Érzékeny Hashelés (LSH) segítségével és k-anonimitást érvényesít.
Kulcskomponensek: PaillierKeyPair, HomomorphicEncryption és SecureDataSampler.

Ellenőrzési Csővezeték Folyamata

Egy gráffrissítés a következő lépéseken megy keresztül az ellenőrzési veremben, mielőtt a mag motorba kerülne:
1. A TypeTheoryEngine ellenőrzi a típusítéletek helyességét (proveTypeJudgment).
2. A FormalVerificationEngine érvényesíti az invariánsokat (verifyGraph) - TheoremProver (Rezolúciós Cáfolat).
3. A SecurityProofEngine elvégzi az Információáramlás Elemzést - Bell-LaPadula/Biba Ellenőrzés.
4. A ZKInferenceProver Következtetési Bizonyítékot generál - ZKProofBundle létrehozva.

---

7.1. FORMÁLIS ELLENŐRZÉSI MOTOR

A Formális Ellenőrzési Motor szigorú matematikai keretrendszert biztosít a SelfSimilarRelationalGraph strukturális, temporális és kvantum integritásának biztosítására. Egy InvariantRegistry-t kombinál a futásidejű tulajdonság-ellenőrzéshez egy TheoremProver-rel és HoareLogicVerifier-rel a statikus és dinamikus formális bizonyítékokhoz. Ez az alrendszer biztosítja, hogy minden gráftranszformáció és következtetési lépés megfeleljen az előre meghatározott biztonsági és helyességi specifikációknak.

FormalVerificationEngine

A FormalVerificationEngine az elsődleges orchestrátor az ellenőrzési veremhez. Integrálja a gráf érvényesítést, a logikai dedukciót és a következtetés monitorozást egy egységes interfészbe.

Kulcsfüggvények:
- verifyGraph(graph): Kimerítő ellenőrzést végez az összes regisztrált invariánson a SelfSimilarRelationalGraph aktuális állapotával szemben. Végigiterál az InvariantRegistry-n és egy boolean értéket ad vissza, amely jelzi a teljes megfelelést.
- prove(proposition): Interfészeli a TheoremProver-t egy adott Proposition formális levezetésének megkísérléséhez. Egy Proof objektumot ad vissza, amely tartalmazza a ProofRule alkalmazások sorozatát.
- verifyInferenceHook(input, output): Speciális visszahívás, amelyet a ChaosCoreKernel használ annak érvényesítésére, hogy egy specifikus következtetési lépés nem sértette meg a rendszer formális korlátait.

Ellenőrzési Előzmények
A motor fenntart egy VerificationHistory naplót, amely minden ellenőrzési kísérlet kronológiai nyilvántartását tárolja, beleértve az időbélyeget, az ellenőrzött InvariantType-ot és a siker/kudarc eredményt.

InvariantRegistry és Predikátumok

Az InvariantRegistry PredicateFn függvények gyűjteményét kezeli, amelyek meghatározzák a gráf "fizikai törvényeit". Minden predikátum egy InvariantType-hoz van társítva.

Alapvető Invariánsok
A rendszer több kritikus predikátumot valósít meg, amelyeket a verifyGraph során használnak:
- Kapcsolódás (BFS): Szélességi Keresést alkalmaz annak biztosítására, hogy a gráf egyetlen összefüggő komponens maradjon, vagy specifikus elérhetőségi követelményeknek feleljen meg.
- Kvantumállapot Normalizálás: Ellenőrzi, hogy bármely QuantumState-ben a négyzetes amplitúdók összege egyenlő 1,0-val (kis epsilon-on belül), biztosítva a fizikai érvényességet.
- Temporális Konzisztencia: Ellenőrzi a HistoryEntry naplókat annak biztosítására, hogy egyetlen TemporalNode verzió sem tartalmaz ellentmondásos időbélyegeket vagy oksági paradoxonokat.

Invariáns Prioritás
Az invariánsokhoz prioritások vannak rendelve az InvariantType.priority() segítségével. A MEMORY_SAFETY (10) és a TYPE_SAFETY (9) ellenőrzése megelőzi az alacsonyabb szintű tulajdonságokat, mint a TEMPORAL_CONSISTENCY (2).

TheoremProver és Logikai Rendszer

A TheoremProver egy szimbolikus érvelési motor, amely Proposition és Term objektumokon működik. Automatizált dedukciót támogat több klasszikus és speciális logikai technika segítségével.

Bizonyítási Mechanizmusok:
- Egyesítés: A motor strukturális egyesítést végez Term változók és konstansok között, hogy megtalálja a logikai kifejezéseket kielégítő helyettesítéseket.
- Visszafelé Láncolás: Egy cél Proposition-ból kiindulva a bizonyító rekurzívan keres ProofRule alkalmazásokat (mint a MODUS_PONENS), amelyek visszavezetnek ismert AXIOM állapotokhoz.
- Rezolúciós Cáfolat: Technika egy állítás bizonyítására azáltal, hogy megmutatja, hogy tagadása ellentmondáshoz vezet.

Memóriakezelés
A Proposition és Term objektumok referenciaszámlálást alkalmaznak a memória hatékony kezeléséhez a nagy bizonyítási fák generálása során, megakadályozva a szivárgásokat a TheoremProver-ben.

HoareLogicVerifier

A HoareLogicVerifier a gráftranszformációk helyességére összpontosít, programokként kezelve azokat előfeltételekkel és utófeltételekkel (Hoare Hármasok: {P} C {Q}).

Axiómák és Szabályok:
- Hozzárendelési Axióma: Érvényesíti egy csomópont tulajdonságának vagy egy qubit állapotának átmenetét közvetlen hozzárendelés esetén.
- Hármas Összetétel: A SEQUENCE_RULE-t alkalmazza több gráfművelet összeláncolásához, biztosítva, hogy az A művelet utófeltétele kielégíti a B művelet előfeltételét.
- Hurok Invariánsok: Iteratív gráf optimalizálások során (mint az ESSO-ban lévők) annak bizonyítására használják, hogy bizonyos tulajdonságok (pl. ENTANGLEMENT fok) stabilak maradnak az iterációk során.

Referencia Táblázatok

Invariáns Típusok és Prioritások

| Invariáns Típus | Prioritás | Leírás |
| :--- | :--- | :--- |
| MEMORY_SAFETY | 10 | Érvényes mutatóhivatkozásokat és határokat ellenőriz a gráfon belül. |
| TYPE_SAFETY | 9 | Biztosítja, hogy a csomópont/él adatok megfelelnek a séma definícióknak. |
| CONNECTIVITY | 8 | Érvényesíti a gráf topológiát BFS segítségével. |
| COHERENCE | 7 | Ellenőrzi a kvantumállapot stabilitási küszöbértékeket. |
| QUANTUM_STATE | 5 | Érvényesíti a komplex amplitúdó normalizálást. |

Elsődleges Bizonyítási Szabályok

| Szabály | Minimális Premisszák | Cél |
| :--- | :--- | :--- |
| AXIOM | 0 | Alap igazság követelmények nélkül. |
| MODUS_PONENS | 2 | Ha P és P => Q, akkor Q. |
| FRAME_RULE | 1 | Bizonyítja, hogy a helyi állapot tulajdonságai igazak maradnak a globális kontextusban. |
| ASSIGNMENT_AXIOM | 0 | Alapvető Hoare Logika szabály az állapotfrissítésekhez. |

---

7.2. TÍPUSELMÉLET ÉS LINEÁRIS ERŐFORRÁS ELLENŐRZÉS

A Típuselmélet és Lineáris Erőforrás Ellenőrzési rendszer, amelyet a type_theory.zig valósít meg, formális keretrendszert biztosít a kvantum-relációs programok strukturális és erőforrás-orientált integritásának biztosítására. Kifinomult típusrendszert valósít meg, amely függő típuselméletén (Pi és Sigma típusok) és lineáris logikán alapul, véges erőforrások, mint kvantumállapotok és hardver-korlátozott változók kezeléséhez.

Típuselmélet Motor

A TypeTheoryEngine az elsődleges orchestrátor a típusítéletek bizonyításához és a típusuniverzum konzisztenciájának fenntartásához. Támogatja a Curry-Howard megfeleltetést, az állításokat típusokként, a bizonyítékokat programokként kezelve.

Kulcsfüggvények:
- proveTypeJudgment: Ellenőrzi, hogy egy adott kifejezés vagy term egy specifikus típust lakja-e egy megadott kontextuson belül.
- proveSubtyping: Meghatározza, hogy az A típus altípusa-e B-nek, támogatva az univerzum polimorfizmust és a variancát a funkcionális típusokban.
- proveEquality: Ellenőrzi a típusok közötti strukturális és definicionális egyenlőséget, normalizált hasheket alkalmazva.

Típushierarchia és Struktúrák

A rendszer megkülönbözteti az alaptípusokat (pl. UNIT, BOOL, NAT) és az összetett kompozit típusokat.

| Kategória | TypeKind | Leírás |
| :--- | :--- | :--- |
| Alap | UNIT, BOOL, NAT, INT, REAL, STRING | Szabványos primitív típusok. |
| Függő | DEPENDENT_FUNCTION (Pi), DEPENDENT_PAIR (Sigma) | Értékektől függő típusok, formális bizonyítékokat lehetővé téve. |
| Lineáris | QUANTUM_TYPE | A LinearTypeChecker által irányított típusok az erőforrás-biztonsághoz. |
| Strukturális | ARRAY, TUPLE, RECORD, SUM | Összetett adataggregátorok. |

Lineáris Erőforrás Ellenőrzés

A LinearTypeChecker erőforrás-korlátokat érvényesít a LinearityMode alapján. Ez kritikus fontosságú a kvantumműveleteknél, ahol a "nem-klónozás" tétel érvényes, vagy a hardver erőforrásokhoz, amelyeket pontosan egyszer kell felhasználni.

Linearitási Módok:
- LINEAR: Pontosan egyszer kell felhasználni.
- AFFINE: Legfeljebb egyszer használható (nulla vagy egy).
- RELEVANT: Legalább egyszer kell felhasználni.
- UNRESTRICTED: Szabványos szemétgyűjtött vagy újrafelhasználható változók.

Megvalósítás: ResourceUsage
A ResourceUsage struktúra nyomon követi, hogy egy kötés hányszor lett elérve a checkLinearity menet során. Ha egy LINEAR erőforrást kétszer érnek el, vagy egy hatókör végén felhasználatlan marad, a motor TypeTheoryError.LinearityViolation hibát jelez.

Kategóriaelmélet Ellenőrzés

A motor speciális logikát tartalmaz annak ellenőrzésére, hogy a megvalósítási struktúrák megfelelnek-e a Kategóriaelmélet törvényeinek. Ezt elsősorban a Funktor és Monad megvalósítások érvényesítésére használják a relációs gráfon belül.

Ellenőrzött Törvények:
1. Funktor Identitás: fmap id == id
2. Funktor Összetétel: fmap (f . g) == fmap f . fmap g
3. Monad Bal Identitás: return a >>= f == f a
4. Monad Jobb Identitás: m >>= return == m
5. Természetes Transzformáció: Ellenőrzi a természetességi négyzetek kommutativitását két funktor között.

Megvalósítási Részletek

Típus Strukturális Egyenlőség
A típusokat TypeKind kombinációjával és paramétereik vagy mezőik rekurzív hashével hasonlítják össze. A Type struktúra fenntart egy hash_cache-t az összehasonlítások gyorsítására.

Függő Típus Kezelés
A Type struktúra bound_variable és body_type mezőket használ a Pi (Π) és Sigma (Σ) típusok megjelenítéséhez.
- Pi Típusok: Függő függvényeket képviselnek, ahol a visszatérési típus a bemeneti értéktől függ.
- Sigma Típusok: Függő párokat képviselnek, ahol a második elem típusa az első értékétől függ.

Hibakezelés
A rendszer átfogó TypeTheoryError hibakészletet alkalmaz a diagnosztikai visszajelzés biztosítására az ellenőrzés során:
- TypeMismatch: A várt típus A, de B-t találtunk.
- UnificationFailure: Nem sikerült közös helyettesítést találni két típushoz.
- CategoryLawViolation: Egy struktúra nem teljesített egy funktor vagy monad törvény ellenőrzést.

---

7.3. BIZTONSÁGI BIZONYÍTÉKOK ÉS INFORMÁCIÓÁRAMLÁS ELEMZÉS

A DANCING keretrendszer Ellenőrzési és Biztonsági Vereme tartalmaz egy dedikált Biztonsági Bizonyíték Motort, amelyet szigorú információáramlási politikák és hozzáférés-vezérlési invariánsok érvényesítésére terveztek a SelfSimilarRelationalGraph-on keresztül. Ez a rendszer rács-alapú biztonsági modelleket, nem-interferencia bizonyítékokat és kriptográfiai struktúrákat alkalmaz annak biztosítására, hogy a kvantum-relációs adatfeldolgozás formális biztonsági tulajdonságoknak feleljen meg.

Biztonsági Rácsok és Szintek

A rendszer a biztonságot és az integritást matematikai rácsokként valósítja meg, lehetővé téve a "dominancia" és az "információáramlás" formális érvelését.

- SecurityLevel: Titkossági rácsot valósít meg (Bell-LaPadula modell) PUBLIC-tól TOP_SECRET-ig terjedő szintekkel.
- IntegrityLevel: Integritási rácsot valósít meg (Biba modell) UNTRUSTED-tól KERNEL-ig terjedő szintekkel.

Rács Műveletek
A SecurityLevel és IntegrityLevel enumok módszereket biztosítanak a rács műveletekhez:
- Join (⊔): Kiszámítja a legkisebb felső korlátot.
- Meet (⊓): Kiszámítja a legnagyobb alsó korlátot.
- Dominancia: Egy L1 szint dominálja L2-t, ha L1 >= L2 a titkossági rácsban.

Biztonsági Bizonyíték Motor

A SecurityProofEngine a biztonsági tulajdonságok ellenőrzésének központi orchestrátoraként működik a gráfon belül. Kezeli a bizonyítékok életciklusát és biztosítja, hogy minden gráfátmenet fenntartsa a meghatározott biztonsági álláspontot.

Kulcsfüggvények

| Függvény | Cél |
| :--- | :--- |
| proveInformationFlowSecurity | Érvényesíti, hogy a csomópontok között mozgó adatok nem sértik a titkossági vagy integritási korlátokat. |
| proveNonInterference | Biztosítja, hogy a magas biztonsági bemenetek ne befolyásolják az alacsony biztonsági kimeneteket, megakadályozva az oldalsó csatornákat. |
| proveAccessControl | Ellenőrzi, hogy egy főszereplőnek megvannak-e a szükséges jogai (RBAC/ABAC/MAC/DAC) egy specifikus művelethez. |
| proveIntegrity | Ellenőrzi, hogy az adatokat nem módosította-e olyan főszereplő, akinek integritási szintje alacsonyabb az adaténál. |

Információáramlás és Nem-Interferencia

A motor két elsődleges formális modellt érvényesít:
1. Bell-LaPadula (Titkosság):
   - Nem Olvas Felfelé: Egy L szintű alany nem olvashat O szintű objektumot, ha O > L.
   - Nem Ír Lefelé: Egy L szintű alany nem írhat O szintű objektumba, ha O < L.
2. Biba (Integritás):
   - Nem Olvas Lefelé: Egy L szintű alany nem olvashat O szintű objektumot, ha O < L (a szennyeződés megelőzéséhez).
   - Nem Ír Felfelé: Egy L szintű alany nem írhat O szintű objektumba, ha O > L.

NonInterferenceProver
A NonInterferenceProver az Unraveling Lemmát és a BisimulationRelation-t alkalmazza annak bizonyítására, hogy a rendszer biztonságos az információszivárgással szemben. Ellenőrzi, hogy bármely két "alacsony-ekvivalens" állapothoz (alacsony jogosultságú megfigyelő számára megkülönböztethetetlen) bármely átmenet szintén alacsony-ekvivalens új állapotokhoz vezet.

Hozzáférés-vezérlés Ellenőrzés

Az AccessControlVerifier több paradigmát támogat a relációs gráfon belüli engedélyek kezeléséhez:
- RBAC (Szerepalapú): Főszereplőkhöz rendelt szerepekhez társított engedélyek.
- ABAC (Attribútumalapú): Az alany, objektum és környezet attribútumain alapuló logika.
- MAC (Kötelező): Rögzített politikák biztonsági engedélyeken alapulva (rács-alapú).
- DAC (Diszkrecionális): Tulajdonos által meghatározott engedélyek.

Kriptográfiai Struktúrák

A bizonyítékok és adatláncok fizikai integritásának biztosítása érdekében a rendszer több kriptográfiai primitívet valósít meg:
- HashChain: Auditnaplókhoz és temporális verziókezeléshez használják, biztosítva, hogy az előzmények ne legyenek manipulálhatók a lánc megszakítása nélkül.
- MerkleTree: Hatékony tagság-ellenőrzést biztosít nagy adatkészletekhez vagy gráf alstruktúrákhoz.
- CommitmentScheme: "Kötelezettségvállalás-majd-felfedés" protokollt valósít meg, amely elengedhetetlen a nulla-tudás bizonyítékokhoz és a biztonságos többfél szinkronizáláshoz.
- cryptographic_binding: Véglegesítési lépés, amely egy biztonsági bizonyítékot egy specifikus gráfállapothoz köt magas erősségű hashek (SHA-256/SHA-512/Blake3) segítségével.

Kulcs Adatstruktúrák

| Típus | Definíció | Szerep |
| :--- | :--- | :--- |
| AccessRight | enum(u8) | Bitmaszk READ, WRITE, EXECUTE, DELETE, ADMIN jogokhoz. |
| SecurityProofsConfig | struct | Konstansok az összefoglaló méretekhez és hozzáférési jog bit pozíciókhoz. |
| SecurityError | hibakészlet | Specifikus hibák, mint IllegalInformationFlow vagy BellLaPadulaViolation. |

---

7.4. NULLA-TUDÁS ELLENŐRZÉS ÉS ELLENŐRZÖTT KÖVETKEZTETÉS

A Nulla-Tudás Ellenőrzés és Ellenőrzött Következtetés alrendszer biztosítja a következtetési eredmények számítási integritásának biztosításához szükséges kriptográfiai infrastruktúrát anélkül, hogy felfedné az alapul szolgáló modellsúlyokat vagy érzékeny bemeneti adatokat. Groth16 zk-SNARK-okat, kötelezettségvállalási sémákat és differenciális adatvédelmet kombinál egy "kötelezettségvállalás-majd-bizonyítás" csővezeték létrehozásához az ellenőrizhető számításhoz.

ZK Ellenőrzési és Bizonyítási Rendszer

Az ellenőrzési verem magja a ZKInferenceProver, amely orchestrálja a nulla-tudás bizonyítékok generálását a Groth16 protokoll segítségével snarkjs és circom alkalmazásával.

Megvalósítás és Életciklus
A rendszer szigorú életciklust követ a bizonyíték generáláshoz és érvényesítéshez, amelyet a ZKProofBundle foglal magában.

1. Áramkör Konfiguráció: A ZKCircuitConfig meghatározza az elérési utakat a .circom fájlokhoz, WASM tanúkhoz és ellenőrzési kulcsokhoz.
2. Tanú Generálás: A lebegőpontos értékek rögzített pontú egészekre skálázódnak a precision_bits segítségével a véges mező aritmetikával való kompatibilitás érdekében.
3. Bizonyíték Generálás: A CircomProver gyermekfolyamatként hajtja végre a snarkjs-t Groth16Proof objektumok generálásához.
4. Ellenőrzés: A bizonyítékokat PublicSignals-szal szemben érvényesítik annak megerősítésére, hogy a kimenet a kötelezettségvállalt bemenetből és modellállapotból lett levezetve.

Ellenőrzött Következtetési Motor

A VerifiedInferenceEngine egy "kötelezettségvállalás-majd-bizonyítás" csővezetéket valósít meg. Biztosítja, hogy egy neurális következtetés (kifejezetten Affin Csatolási rétegeket alkalmazva) minden lépése rögzítésre kerüljön egy ProofOfCorrectness nyomban.

Kulcskomponensek:
- Affin Csatolási Rétegek: A layer_weights_s (skála) és layer_weights_t (eltolás) tenzorokon keresztül valósítva meg.
- Kötelezettségvállalási Séma: Blake3 és Sha256 hibridjét alkalmazza kötelező kötelezettségvállalások létrehozásához a bemenetekhez és kimenetekhez.
- Differenciális Adatvédelem: A DifferentialPrivacy struktúra Gauss vagy Laplace zajt alkalmaz a kimenetekre a tagság-következtetési támadások megelőzéséhez.
- Biztonságos Aggregálás: Adatvédelem-megőrző modellfrissítéseket tesz lehetővé több résztvevő között.

Ellenőrzött Következtetési Csővezeték

| Szakasz | Entitás | Függvény |
| :--- | :--- | :--- |
| Kötelezettségvállalás | CommitmentScheme | StoredCommitment-et generál az x bemenethez. |
| Végrehajtás | VerifiedInferenceEngine | Előre menetet számít N rétegen keresztül. |
| Nyom | ProofOfCorrectness | Naplózza a közbenső aktiválásokat és réteg hasheket. |
| Bizonyíték | ZKInferenceProver | SNARK-ot generál a végrehajtási nyomhoz. |
| Ellenőrzés | BatchVerifier | Több bizonyítékot érvényesít egyidejűleg. |

Kriptográfiai Primitívek

A rendszer több speciális kriptográfiai struktúrára támaszkodik, amelyeket a zk_verification.zig definiál:

Kötelezettségvállalás és Tagság:
- CommitmentScheme: Pedersen és Hash alapú kötelezettségvállalásokat támogat. Blake3-at alkalmaz a nagy teljesítményű hasheléshez és Sha256-ot az áramkör kompatibilitáshoz.
- MembershipProof: Merkle Fa megvalósítást alkalmaz annak bizonyítására, hogy egy specifikus adatpont létezik egy kötelezettségvállalt adatkészleten belül anélkül, hogy magát a pontot felfedné.

Aláírás és Tartomány Bizonyítékok:
- SchnorrSignature: Nem-formálható aláírásokat biztosít a következtetési szolgáltató identitásának ellenőrzéséhez.
- RangeProof: Bit-dekompozíciót valósít meg annak bizonyítására, hogy egy v érték a [0, 2^n - 1] tartományban van anélkül, hogy v-t felfedné. Ez kritikus fontosságú annak biztosításához, hogy a rögzített pontú értékek ne csordulják túl a ZK áramkör prím mezőjét.

Differenciális Adatvédelem
A DifferentialPrivacy modul kezeli az adatvédelmi keretet (ε, δ) és biztosítja:
- addGaussianNoise: Perturbálja a kimeneti vektort a következtetési függvény érzékenysége alapján.
- addLaplaceNoise: Nehéz farkú zaj követelményekhez használják specifikus relációs lekérdezéseknél.

Kötegelt Ellenőrzés és Aggregálás

A nagy átviteli sebességű következtetési kérések kezeléséhez a rendszer BatchVerifier-t és ProofAggregator-t alkalmaz.
- BatchVerifier: Több ZKProofBundle objektumot gyűjt össze és egyetlen műveletben ellenőrzi azokat a terhelés csökkentéséhez.
- ProofAggregator: Merkle befoglalási bizonyíték stratégiát alkalmaz. Bizonyítékok gyűjteményét veszi, Merkle fát épít hashjeikből, és egyetlen gyökér kötelezettségvállalást állít elő. Ez lehetővé teszi egy ügyfél számára N következtetés érvényességének ellenőrzését csupán a Merkle gyökér és az összesített bizonyíték ellenőrzésével.

---

7.5. ADATKÉSZLET ELHOMÁLYOSÍTÁS ÉS ADATVÉDELMI KERETRENDSZER

Az Adatkészlet Elhomályosítás és Adatvédelmi Keretrendszer biztosítja a DANCING ökoszisztémán belüli adatvédelem, integritás és izoláció biztosításához szükséges kriptográfiai és statisztikai primitíveket. Homomorf titkosítást valósít meg a biztonságos számításhoz, k-anonimitást az adatmintavételhez és Lokalitás-Érzékeny Hashelést (LSH) az adatvédelem-megőrző hasonlóság-elemzéshez.

1. Homomorf Titkosítási Alrendszer

A keretrendszer megvalósítja a Paillier kriptoszisztémát, egy additív homomorf titkosítási sémát, amely lehetővé teszi az összeadásokat és skaláris szorzásokat a titkosított adatokon visszafejtés nélkül.

Paillier Kulcsgenerálás
A PaillierKeyPair generálás a Miller-Rabin primalitástesztre támaszkodik a nagy p és q prímek azonosításához.
- Prímgenerálás: A generatePrime128 128 bites prímjelölteket hoz létre a crypto.random.bytes segítségével és az isProbablePrime128 segítségével ellenőrzi azokat.
- Primalitástesztelés: Az isProbablePrime128 megvalósítja a Miller-Rabin algoritmust determinisztikus tanúk készletével u128 tartományokhoz.
- Kulcskomponensek:
  - n: p és q szorzata.
  - g: A generátor, általában n+1.
  - λ: p-1 és q-1 legkisebb közös többszöröse.
  - μ: L(g^λ mod n²) moduláris multiplikatív inverze, ahol L(x) = (x-1)/n.

Biztonságos Műveletek
A HomomorphicEncryption struktúra a következő alapvető műveleteket biztosítja:

| Függvény | Leírás | Megvalósítás |
| :--- | :--- | :--- |
| encrypt | Titkosítja az m nyílt szöveget a (n, g) nyilvános kulcs és véletlen r segítségével. | c = g^m · r^n mod n² |
| decrypt | Visszafejti a c titkosított szöveget a (λ, μ) privát kulcs segítségével. | m = L(c^λ mod n²) · μ mod n |
| add | Homomorfikusan összeadja a c1, c2 titkosított szövegeket. | c_sum = c1 · c2 mod n² |
| multiplyScalar | Megszorozza a titkosított c-t egy k nyílt szöveges skalárral. | c_prod = c^k mod n² |

2. Adatkészlet Ujjlenyomat és LSH Indexelés

A DatasetFingerprint rendszer lehetővé teszi az adatkészletek azonosítását és összehasonlítását, miközben megőrzi az egyéni rekordok adatvédelmét Lokalitás-Érzékeny Hashelés (LSH) segítségével.

LSH Mechanizmus
A rendszer a magas dimenziós adatokat alacsony dimenziós "vödrökbe" képezi le, ahol a hasonló elemek közel maradnak egymáshoz.
- Ujjlenyomat Generálás: Blake3 és Sha256 segítségével stabil azonosítókat hoz létre adatdarabokhoz.
- Hasonlóság Pontozás: Hamming távolságot alkalmaz két DatasetFingerprint példány hasonlóságának kiszámításához, lehetővé téve a deduplikálást vagy kapcsolatleképezést a nyers adatok felfedése nélkül.

3. Biztonságos Adatmintavevő és Adatvédelmi Keretszabályozás

A SecureDataSampler biztosítja, hogy az elemzéshez a rendszerből kinyert adatok megfeleljenek a k-anonimitási és differenciális adatvédelmi korlátoknak.

Adatvédelmi Mechanizmusok:
- k-anonimitás: Biztosítja, hogy egy kiadott mintában bármely egyéni rekord megkülönböztethetetlen legyen legalább k-1 más egyéntől.
- Differenciális Adatvédelmi Keret: Nyomon követi a kumulatív "adatvédelmi veszteséget" (epsilon) több lekérdezésen keresztül. A SecureDataSampler fenntart egy keretet; ha az kimerül, a további nagy pontosságú lekérdezések blokkolásra kerülnek a rekonstrukciós támadások megelőzése érdekében.
- Zajinjektálás: Laplace vagy Gauss zajt ad az összesített eredményekhez a CryptoConfig.NOISE_BITS paraméter alapján.

4. Adatkészlet Izoláció és Hozzáférés-vezérlés

A DatasetIsolation modul szigorú határokat érvényesít a különböző adatsilók között korlátok és sebességkorlátozás segítségével.

Izolációs Komponensek:
- Hozzáférési Politikák: Meghatározza, hogy ki férhet hozzá specifikus DatasetFingerprint tartományokhoz.
- Sebességkorlátozás: Megakadályozza az oldalsó csatornás támadásokat (mint az időzítési támadások) a visszafejtési vagy hasonlósági kérések gyakoriságának korlátozásával.
- Memória Fertőtlenítés: A keretrendszer a secureZero-t alkalmazza az érzékeny kriptográfiai anyagok (mint a PaillierKeyPair komponensek) memóriából való törlésére közvetlenül a használat után vagy a felszabadításkor.

Ellenőrzés a ProofOfCorrectness Segítségével
Annak biztosítására, hogy az elhomályosítás nem rontotta el az alapul szolgáló relációs logikát, a ProofOfCorrectness struktúra Merkle gyökér kötelezettségvállalást generál. Ez a kötelezettségvállalás lehetővé teszi egy harmadik fél számára annak ellenőrzését, hogy egy specifikus transzformációt alkalmaztak az adatkészletre anélkül, hogy magát az adatot látná.

5. Kulcsfüggvények Összefoglalása

| Függvény / Struktúra | Szerep |
| :--- | :--- |
| PaillierKeyPair.generate() | n, g, λ, μ generálása Miller-Rabin segítségével. |
| HomomorphicEncryption.add() | Titkosított összeadást végez moduláris szorzással. |
| isProbablePrime128() | Miller-Rabin megvalósítás 128 bites prímekhez. |
| modInverse256() | Moduláris inverzt számít a Mu (μ) generáláshoz. |
| DatasetFingerprint | LSH-alapú hasonlóság és integritás nyomon követő. |
| secureZero() | Konstans idejű memóriatörlés érzékeny kulcsokhoz. |

---

8. BIZTONSÁGI PRIMITÍVEK ÉS KERESZTMETSZŐ SEGÉDPROGRAMOK

A Biztonsági Primitívek és Keresztmetsző Segédprogramok réteg biztosítja a DANCING keretrendszer integritásának, biztonságának és együttműködési képességének biztosításához szükséges alapvető infrastruktúrát. Ez a réteg két elsődleges területre oszlik: szigorú futásidejű biztonsági ellenőrzések a safety.zig-en keresztül és globális rendszer-orchestráció a mod.zig regiszteren keresztül.

Alapvető Architektúra Kontextus

Hibrid kvantum-relációs keretrendszerként a DANCING determinisztikus biztonsági garanciákat igényel a magas szintű relációs logika és az alacsony szintű hardver absztrakciók közötti áthidaláskor. Az itt definiált segédprogramokat minden főbb alrendszer használja, a CREVPipeline-tól a QuantumLogic motorig, biztosítva, hogy a memóriaműveletek, numerikus konverziók és véletlenszám-generálás megfeleljenek a kriptográfiai és biztonsági szabványoknak.

---

8.1. BIZTONSÁGI PRIMITÍVEK (safety.zig)

A safety.zig modul határellenőrzött műveletek és kriptográfiai segédprogramok készletét valósítja meg, amelyeket a memória- és aritmetikai hibák általános osztályainak megelőzésére terveztek. Bevezeti a SafetyError készletet, amely részletes hibajelentést biztosít a túlcsordulásokhoz, null mutatókhoz és igazítási sértésekhez.

Hibakezelés és Konvertálás

A modul SafetyError készletet definiál a futásidejű sértések kezeléséhez, amelyek nem definiált viselkedéshez vagy biztonsági sebezhetőségekhez vezethetnek.

| Hiba | Leírás |
| :--- | :--- |
| IntegerOverflow | Egy művelet vagy konvertálás eredménye meghaladja a céltype maximális értékét. |
| IntegerUnderflow | Egy művelet vagy konvertálás eredménye kisebb a céltype minimális értékénél. |
| InvalidPointer | A megadott típus nem mutató vagy opcionális mutató. |
| NullPointer | Null mutatón kíséreltek meg műveletet. |
| MisalignedPointer | Egy mutató nem teljesíti a céltype igazítási követelményeit. |
| BufferTooSmall | Egy célpuffer nem rendelkezik elegendő kapacitással egy másolási vagy szelet-művelethez. |

Biztonságos Konvertálási Műveletek:
- safeIntCast(comptime T, value): Ellenőrzött egész szám konvertálást végez. Érvényesíti az előjelességet és a bit-szélességet a csendes túlcsordulások vagy alulcsordulások megelőzéséhez, mielőtt végrehajtja az @intCast-ot.
- safePtrCast(comptime T, ptr): Biztosítja, hogy egy mutató nem null és megfelelően igazított a T céltype számára, mielőtt végrehajtja az @alignCast és @ptrCast műveleteket.
- validatePointer(ptr): Nem-dobó segédprogram annak ellenőrzésére, hogy egy mutató (vagy opcionális mutató) nem null és érvényes címmel rendelkezik.

Biztonságos Entrópia: SecureRng

A SecureRng struktúra hibrid entrópiaforrást biztosít. Kombinálja a rendszer kriptográfiai véletlenszám-generátorát (std.crypto.random) egy tartalék Multiplikatív Kongruenciális Generátorral (MCG) a rendelkezésre állás és a további keverés biztosítása érdekében.

Az entrópia adatfolyam:
A SecureRng XOR-alapú keverési stratégiát alkalmaz. Minden kéréshez előre lépteti a belső fallback_state-et egy 64 bites LCG/MCG konstans segítségével.

Kulcsmódszerek:
- intRange(T, min, max): Elutasítás-mintavételezést valósít meg, hogy torzítatlan véletlenszámokat biztosítson egy specifikus tartományon belül.
- float(T): Egyenletesen elosztott lebegőpontos számokat generál a [0, 1) tartományban az f32 vagy f64 mantissza bitjeinek manipulálásával.

Memória és Időzítési Biztonság

A modul konstans idejű műveleteket és biztonságos memóriatörlést biztosít az oldalsó csatornás támadások és adatszivárgás megelőzéséhez.

- secureZeroBytes(slice): Illékony memória-hozzáférést alkalmaz annak biztosítására, hogy a fordítóprogram ne optimalizálja el az érzékeny pufferek (pl. privát kulcsok vagy kvantumállapotok) nullázását.
- secureCompare(a, b): Konstans idejű összehasonlítást végez két bájt-szelet között. A végrehajtási idő csak a szeletek hosszától függ, nem a tartalomtól, megakadályozva az időzítési támadásokat.
- MonotonicClock: Az std.time.nanoTimestamp köré épített burkoló, amely inicializálási ponthoz viszonyított eltelt időt biztosít, enyhítve a rendszeróra-ugrások problémáit.

BigInt512 Aritmetika

A kriptográfiai kötésekhez és kvantumállapot-ellenőrzéshez szükséges moduláris aritmetikához a safety.zig megvalósítja a BigInt512-t. Ez a struktúra 512 bites előjel nélküli egész számokat kezel nyolc u64 "végtagból" álló tömb segítségével.

BigInt512 Megvalósítási Részletek:
- add(a, b): Végtagról végtagra összeadást végez átvitel-propagálással az std.math.add segítségével.
- mul(a, b): 512 bites szorzást valósít meg az 1024 bites közbenső szorzat kiszámításával és az alsó 512 bitre való csonkítással.
- modExp(base, exp, mod): Moduláris hatványozást biztosít, amely elengedhetetlen a BigInt512 alapú kötelezettségvállalások és bizonyítékok ellenőrzéséhez.

Puffer Műveletek

A biztonság kiterjed a szelet-kezelésre és az adatmozgatásra explicit határellenőrzésen keresztül.

- safeSlice(T, data, start, end): Érvényesíti, hogy a start és end indexek a data szelet határain belül vannak, mielőtt visszaad egy részszeletét.
- safeCopy(T, dest, src): Ellenőrzi, hogy a célpuffer elég nagy a forrásadatok tárolásához, mielőtt végrehajtja az std.mem.copyForwards-ot. SafetyError.BufferTooSmall hibát ad vissza kudarc esetén.

---

8.2. MODULREGISZTER ÉS RENDSZERINTEGRÁCIÓ (mod.zig)

A mod.zig fájl a DANCING keretrendszer központi orchestrációs csomópontjaként és nyilvános interfészeként szolgál. Modulregiszterként működik, amely importál minden belső alrendszert - az alapvető gráflogikától a kvantumhardver absztrakcióig és az ellenőrzési motorokig - és újraexportálja elsődleges típusaikat és függvényeiket egy egységes névtérbe. Ez az architektúra lehetővé teszi a kódbázis többi részének és a külső fogyasztóknak, hogy egyetlen, összefüggő belépési ponton keresztül lépjenek kapcsolatba a rendszerrel.

Rendszerhierarchia és Rétegzett Architektúra

Az integráció rétegzett megközelítést követ, ahol a magas szintű futtatókörnyezetek koordinálják a speciális végrehajtási motorokat és hardver absztrakciókat.

1. Orchestrációs Réteg: A ZRuntime és a CREVPipeline kezeli a magas szintű állapotot, a változók életciklusát és az adatbetöltést.
2. Logika és Optimalizálás Réteg: A RelationalQuantumLogic, az EntangledStochasticSymmetryOptimizer (ESSO) és a ReasoningOrchestrator kezeli a relációs adatok transzformálását és optimalizálását.
3. Alapvető Adatréteg: A SelfSimilarRelationalGraph, a TemporalGraph és az FNDSManager (Fraktál Csomópont Adatrendszer) biztosítja az információ alapvető reprezentációját.
4. Végrehajtási és Hardver Réteg: A ChaosCoreKernel, a RelationalGraphProcessingUnit (RGPU) és a QuantumHardware absztrakciók interfészelnek a fizikai vagy szimulált számítási erőforrásokkal.

Adatfolyam és Csővezeték Integráció

A rendszeren belüli elsődleges adatfolyam nyers relációs bemenettel kezdődik és kinyerési, tárolási és optimalizálási fázisokon halad keresztül.

1. Betöltés: A CREVPipeline RelationalTriplet adatokat kinyeri, tokenizálást és mintaillesztést végezve.
2. Memóriakezelés: A kinyert adatok átkerülnek a SurpriseMemoryManager-hez (a mod.zig-en keresztül integrálva, de a surprise_memory.zig-ben definiálva), amely az újdonság és a Jaccard különbözőség alapján határozza meg a megőrzést.
3. Állapot Reprezentáció: Az adatok a TemporalGraph-ban tárolódnak, ahol NodeVersion és EdgeVersion segítségével verziókezeltek.
4. Futásidejű Végrehajtás: A ZRuntime koordinálja a műveleteket ezeken a struktúrákon keresztül. Aktiválhat ExecutionAction típusokat, mint a relational_operation vagy a quantum_circuit.

A ZRuntime Koordinátor

A ZRuntime a mod.zig által exportált elsődleges magas szintű koordinátor. Fenntartja a SystemState-et és kezeli a ZVariable példányokat, amelyek mindegyike saját SelfSimilarRelationalGraph-ot és RelationalQuantumLogic állapotot foglal magában.

| Entitás | Cél | Kulcsfunkcionalitás |
| :--- | :--- | :--- |
| ZRuntime | Legfelső szintű rendszerkezelő | Orchestrálja a változókat, kifejezéseket és áramköröket. |
| ZVariable | A relációs állapot atomi egysége | Gráfot és kvantumlogikát tart egy specifikus nevű entitáshoz. |
| ExecutionAction | Futásidejű műveleti napló | Kategorizálja a műveleteket, mint az entangle_variables vagy a fractal_transform. |
| RelationalOperationType | Logikai kapuk | Relációs logikát definiál, mint az AND, OR, XOR és ENTANGLE. |

Hardver és Alacsony Szintű Integráció

A mod.zig regiszter összeköti az alapvető logikát a nagy teljesítményű végrehajtási egységekkel és kvantum backendekkel.

- ChaosCoreKernel: Biztosítja az alapvető végrehajtási környezetet, kezelve a ContentAddressableStorage-t (CAS) és a DynamicTaskScheduler-t.
- Relációs GPU (RGPU): Speciális processzor gráfműveletekhez, AsynchronousNoC-ot (Chip-hálózat) alkalmazva az üzenetirányításhoz a feldolgozó magok között.
- Vektor Feldolgozó Egység (VPU): SIMD-gyorsított numerikus műveleteket kezel, amelyek spektrális beágyazásokhoz és mátrix matematikához szükségesek.
- Kvantum Integráció: Áthidalja a magas szintű QuantumCircuit definíciókat a specifikus backendekhez a QuantumTaskAdapter és az IBMQuantumClient segítségével.

Exportált Alrendszerek Összefoglalása

A regiszter a következő főbb névtereket exportálja, lapos hozzáférési mintát biztosítva összetett hierarchikus rendszerekhez:

| Névtér | Fájl | Elsődleges Felelősség |
| :--- | :--- | :--- |
| nsir_core | nsir_core.zig | Alapgráf és qubit primitívek. |
| quantum_logic | quantum_logic.zig | Kvantumállapot szimuláció és kapu logika. |
| z_runtime | z_runtime.zig | Változókezelés és végrehajtás orchestráció. |
| esso_optimizer | esso_optimizer.zig | Sztochasztikus gráf optimalizálás (Szimulált Hőkezelés). |
| crev_pipeline | crev_pipeline.zig | Tudáskinyerés és betöltés. |
| chaos_core | chaos_core.zig | Feladatütemezés és memória kernel. |
| r_gpu | r_gpu.zig | Párhuzamos gráffeldolgozás hardver absztrakció. |
| formal_verification | formal_verification.zig | Invariáns ellenőrzés és tételbizonyítás. |

---

9. SZÓJEGYZÉK

Ez az oldal definíciókat biztosít a DANCING keretrendszerben használt tartomány-specifikus terminológiához, technikai zsargonhoz és architektúrális fogalmakhoz. A kódbázis ötvözi a relációs gráfelméletet a kvantumállapot-szimulációval és a fraktál adatstruktúrákkal, ami pontos szókincset igényel a megvalósítás navigálásához.

Alapvető Tartomány Fogalmak

Relációs Kvantumlogika (RQL)
Hibrid logikai rendszer, amely kiterjeszti a szabványos Boolean műveleteket (AND, OR, XOR) a kvantum tartományba azáltal, hogy QuantumState amplitúdókon működik. Lehetővé teszi a relációs változók összefonódását és a fáziseltolt logika terjedését egy gráfon keresztül.
- Megvalósítás: RelationalQuantumLogic struktúra a quantum_logic.zig-ben.
- Kulcsfüggvények: applyGate, entangle.

Önhasonló Relációs Gráf (SSRG)
A rendszer alapvető adatstruktúrája. Irányított gráfot képvisel, ahol a csomópontok kvantumállapotokat (Qubit) tartalmaznak és az élek "minőségekkel" rendelkeznek (pl. entangled, fractal). A gráf "önhasonló", mert támogatja a fraktál beágyazást és a hierarchikus optimalizálást.
- Megvalósítás: SelfSimilarRelationalGraph struktúra az nsir_core.zig-ben.
- Adatfolyam: Az adatok Node és Edge entitásokként lépnek be, amelyeket ezután topology_hash-be hashelnek az integritás ellenőrzéséhez.

Meglepetési Metrikák
Újdonság-detektálási keretrendszer a memória megőrzés prioritizálásához. A szabványos LRU (Legkevésbé Nemrég Használt) kiürítés helyett a rendszer "Meglepetést" számít a tartalom különbözőség és a temporális újdonság alapján.
- Megvalósítás: SurpriseMetrics struktúra a surprise_memory.zig-ben.
- Képlet komponensek: Jaccard különbözőség, tartalomhash távolság és temporális újdonság.

Technikai Kifejezések és Rövidítések

| Kifejezés | Definíció | Kódmutató |
| :--- | :--- | :--- |
| CAS | Tartalom-Címezhető Tárolás. Rendszer, ahol az adatokat SHA-256 hashük alapján kérik le, nem memóriacím alapján. | ContentAddressableStorage (chaos_core.zig) |
| CREV | Kollaboratív Relációs Kinyerés és Érvényesítés. A csővezeték, amely nyers szöveget (Alany, Reláció, Tárgy) hármasokká alakít. | crev_pipeline.zig |
| ESSO | Összefonódott Sztochasztikus Szimmetria Optimalizáló. Szimulált hőkezelési optimalizáló, amely megőrzi a kvantumkoherenciát és a geometriai szimmetriát. | EntangledStochasticSymmetryOptimizer (c_api.zig) |
| FNDS | Fraktál Csomópont Adatrendszer. Tárolási réteg hierarchikus, skálainvariáns adatmintákhoz. | fnds.zig |
| Qubit | Egy kétállapotú kvantumrendszer reprezentációja komplex a és b amplitúdók segítségével. | Qubit (nsir_core.zig) |
| RGPU | Relációs Gráf Feldolgozó Egység. Szimulált hardver háló párhuzamos gráfbejáráshoz és üzenetküldéshez. | r_gpu.zig |
| VPU | Vektor Feldolgozó Egység. Numerikus motor, amelyet SIMD mátrixműveletekre és Logaritmikus Számrendszer (LNS) matematikára optimalizáltak. | vpu.zig |

Részletes Szójegyzék Bejegyzések

Élminőség (Edge Quality)
Felsorolás, amely meghatározza a SelfSimilarRelationalGraph két csomópontja közötti kapcsolat fizikai vagy logikai állapotát.
- superposition: Az él több potenciális állapotban létezik.
- entangled: Ennek az élnek az állapota egy másik éllel van összekapcsolva.
- fractal: Az él a gráf különböző skálái közötti kapcsolatot képvisel.
- Forrás: EdgeQuality (nsir_core.zig)

Formális Ellenőrzési Invariánsok
Predikátumok, amelyeknek igaznak kell lenniük ahhoz, hogy a gráf "érvényesnek" minősüljön a FormalVerificationEngine-en belül. Ezek magukban foglalják a kapcsolódás ellenőrzéseket és a kvantum normalizálási követelményeket.
- Megvalósítás: InvariantRegistry a formal_verification.zig-ben.
- Hívási hely: verifyGraph.

Paillier Kriptoszisztéma
Részleges homomorf titkosítási séma, amelyet a dataset_obfuscation.zig-ben használnak a titkosított értékek összeadásának lehetővé tételéhez visszafejtés nélkül.
- Megvalósítás: PaillierKeyPair és HomomorphicEncryption (dataset_obfuscation.zig).
- Kulcsfüggvények: encrypt, add.

Z-Változó (Z-Variable)
Magas szintű relációs változó, amelyet a ZRuntime kezel. Minden változó saját SelfSimilarRelationalGraph-ot és RelationalQuantumLogic kontextust foglal magában, lehetővé téve a hatókörözött kvantum-relációs végrehajtást.
- Megvalósítás: ZVariable a z_runtime.zig-ben.
- Életciklus: assign -> relateTo -> measure.

