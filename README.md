<html lang="fr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Roue interactive – avec sons et couleurs alternées</title>
<style>
  :root { --bg:#f7f7f7; --fg:#111; --ring:#e5e7eb; }
  html,body{margin:0;height:100%;background:var(--bg);color:var(--fg);font-family:Helvetica,Arial,sans-serif;}
  .wrap{min-height:100%;display:grid;place-items:center;padding:16px;}
  .container{width:min(92vw,650px);display:grid;gap:10px;justify-items:center;}
  h1{margin:0;font-size:clamp(14px,2.4vw,18px);font-weight:700;letter-spacing:.2px;}
  .board{background:#fff;border:1px solid var(--ring);border-radius:10px;padding:10px;display:grid;gap:8px;justify-items:center;}
  .wheel-area{position:relative;width:100%;display:grid;place-items:center;}
  .pointer{
    position:absolute;top:-2px;left:50%;transform:translateX(-50%) rotate(180deg);
    width:0;height:0;border-left:12px solid transparent;border-right:12px solid transparent;border-bottom:20px solid #1f2937;
    z-index:3;
  }
  canvas{width:min(58vw,540px);max-width:540px;aspect-ratio:1/1;display:block;border-radius:50%;background:#fafafa;box-shadow:inset 0 0 0 5px #f0f0f0;}
  .controls{display:flex;gap:8px;align-items:center;justify-content:center;flex-wrap:wrap;}
  button.primary{background:#111;color:#fff;border:none;border-radius:8px;padding:8px 12px;font-size:12px;font-weight:700;cursor:pointer;}
  button.primary:disabled{opacity:.5;cursor:not-allowed;}
  .legend{font-size:9px;opacity:.7;}
  .overlay{
    position:absolute;inset:0;display:none;align-items:center;justify-content:center;
    z-index:2;
  }
  .overlay .bubble{
    position:relative;max-width:78%;padding:16px 18px;border-radius:12px;background:#111;color:#fff;
    box-shadow:0 8px 24px rgba(0,0,0,.25);
    font-size:clamp(16px,2.8vw,24px);font-weight:700;text-align:center;line-height:1.25;
  }
  .overlay .close{
    position:absolute;top:6px;right:8px;width:26px;height:26px;border-radius:50%;
    border:1px solid rgba(255,255,255,.25);background:rgba(255,255,255,.08);color:#fff;
    display:grid;place-items:center;cursor:pointer;font-size:16px;line-height:1;
  }
</style>
</head>
<body>
<div class="wrap">
  <div class="container">
    <h1>modèle social français</h1>
    <div class="board">
      <div class="wheel-area">
        <div class="pointer"></div>
        <canvas id="wheel" width="800" height="800"></canvas>
        <div id="overlay" class="overlay">
          <div class="bubble">
            <div id="overlayText"></div>
            <div id="overlayClose" class="close" title="Fermer">×</div>
          </div>
        </div>
      </div>
      <div class="controls">
        <button id="spinBtn" class="primary">Tourner la roue</button>
        <span class="legend" id="countInfo"></span>
      </div>
    </div>
  </div>
</div>

<script>
/* -------- Configuration -------- */
let ENTRIES = ["Droits agricoles Recette : 179,7 millions d'euros Date de création : 1962", "Droits de douane Recette : 1605,3 millions d'euros Date de création : 1970", "Taxe à la production sur le quota de sucre, le quota d'isoglucose et le quota de sirop d'inuline Recette : 306,4 millions d'euros Date de création : 1970", "Cotisation assise sur le montant total des honoraires facturés par les commissaires aux comptes lorsqu'ils certifient les comptes ou les informations en matière de durabilité    Date de création : 2017", "Participation des employeurs à l'effort de construction (TPS-PEEC)    Date de création : 1943", "Cotisation HLM et SEM", "Cotisation versée par les organismes HLM et les SEM", "Cotisation additionnelle versée par les organismes HLM et les SEM", "Redevance pour la rémunération pour copie privée Recette : 197 millions d'euros Date de création : 1985", "Taxe de protection des obtentions végétales Recette : 1 millions d'euros Date de création : 1970", "Redevance perçue sur formalités de l'Institut national de la propriété industrielle", "Contribution sur les abondements des employeurs aux plans d'épargne pour la retraite collectifs Recette : 5,3 millions d'euros Date de création : 2001", "Contribution sur les avantages de préretraite d'entreprise", "Contribution sur les régimes de retraite conditionnant la constitution de droits à prestations à l'achèvement de la carrière du bénéficiaire dans l'entreprise", "Contribution sur les indemnités de mise à la retraite Recette : 39 millions d'euros Date de création : 2007", "Contributions patronales et salariales sur les attributions d'options (stock-options) de souscription ou d'achat des actions et sur les attributions gratuites", "Forfait social", "Contribution salariale sur les carried-interests Recette : 2 millions d'euros Date de création : 2010", "Contribution vente en gros Recette : 265 millions d'euros Date de création : 1991", "contributions taux « Lv/Lh »    Date de création : 1999", "Contribution sur les dépenses de promotion des médicaments Recette : 25 millions d'euros Date de création : 2005", "contribution sur le chiffre d'affaires", "Cotisation spéciale sur les boissons alcooliques Recette : 700 millions d'euros Date de création : 1983", "Droits de plaidoirie Recette : 9,8 millions d'euros Date de création : 1921", "Cotisations des employeurs au FNAL", "Fraction de Taxe de solidarité additionnelle", "Contribution sociale de solidarité des sociétés (C3S)    Date de création : 1992", "Droit départemental de passage sur les ouvrages d'art reliant le continent aux îles maritimes Recette : 1 millions d'euros Date de création : 1995", "Contribution additionnelle de solidarité pour l'autonomie (CASA)    Date de création : 2013", "Contribution à la vie étudiante et de campus    Date de création : 2018", "Redevance proportionnelle sur l'énergie hydraulique Recette : 0,9 millions d'euros Date de création : 1919", "Droit de visa de régularisation, taxe de renouvellement du titre de séjour, taxe applicable aux documents de circulation pour étrangers mineurs et taxe perçue à l'occasion de la délivrance du premier titre de séjour", "Contribution forfaitaire représentative des frais de réacheminement", "Redevance perçue à l'occasion de l'introduction des familles étrangères en France", "Redevance pour pollution de l'eau d'origine non domestique Recette : 107 millions d'euros Date de création : 2006", "Redevances pour pollutions diffuses Recette : 107 millions d'euros Date de création : 1964", "Redevance pour stockage d'eau en période d'étiage Recette : 1,3 millions d'euros Date de création : 2006", "Redevance pour protection du milieu aquatique Recette : 8,5 millions d'euros Date de création : 1941", "Redevances de l'eau dans les départements d'outre-mer Recette : 2 millions d'euros Date de création : 2006", "Redevance pour délivrance initiale du permis de chasse", "Redevances cynégétiques", "Redevance de mise sur le marché des substances actives biocides", "Participation pour voirie et réseaux    Date de création : 2000", "Redevance pour création de bureaux ou de locaux de recherche en région Ile-de-France Recette : 124,5 millions d'euros Date de création : 1960", "Fraction des produits annuels de la vente de biens confisqués", "Contributions au Fonds de garantie des assurances obligatoires de dommages (FGAO)    Date de création : 1951", "Contribution au Fonds de garantie des victimes des actes de terrorisme et d'autres infractions    Date de création : 1986", "Contribution au fonds de garantie des dommages consécutifs à des actes de prévention, de diagnostic ou de soins dispensés par des professionnels de santé Recette : 0,9 millions d'euros Date de création : 2011", "Prélèvement \"assurance frontière\" automobile Recette : 0 millions d'euros Date de création : 2007", "Droit de francisation et de navigation en Corse, Droit de passeport en Corse Recette : 3 millions d'euros Date de création : 1994", "Droit de francisation et de navigation Recette : 39,2 millions d'euros Date de création : 1967", "Droit de passeport Recette : 2,2 millions d'euros Date de création : 1967", "Taxe intérieure de consommation sur les produits énergétiques (TICPE) Recette : 24500 millions d'euros Date de création : 1928", "Contribution au service public de l'électricité (CSPE)    Date de création : 2003", "Taxe générale sur les activités polluantes - matériaux d'extraction Recette : 72,5 millions d'euros Date de création : 1999", "Taxe générale sur les activités polluantes - émissions polluantes Recette : 21,6 millions d'euros Date de création : 1998", "Taxe générale sur les activités polluantes - installations classées Recette : 25 millions d'euros Date de création : 1999", "Taxe intérieure sur les houilles, les lignites et les cokes (TICHLC) Recette : 7,6 millions d'euros Date de création : 2006", "Taxe générale sur les activités polluantes - lessives Recette : 44,4 millions d'euros Date de création : 1999", "Taxe générale sur les activités polluantes (TGAP)    Date de création : 2000", "Taxe spéciale sur certains véhicules routiers", "Taxe due par les entreprises de transport public aérien et maritime (Outre-Mer) Recette : 9,4 millions d'euros Date de création : 1994", "Taxe sur les passagers maritimes embarqués à destination d'espaces naturels protégés Recette : 2,6 millions d'euros Date de création : 1995", "Redevance relative aux contrôles renforcés à l'importation des denrées alimentaires d'origine non animale Recette : 3,4 millions d'euros Date de création : 1998", "Redevance relative aux contrôles renforcés à l'importation des denrées alimentaires d'origine non animale Recette : 0 millions d'euros Date de création : 2011", "Redevances d'usage des fréquences radioélectriques (part ANFR) Recette : 15,8 millions d'euros Date de création : 1993", "Droit dû par les entreprises ferroviaires pour l'autorité de régulation des activités ferroviaires", "Droit de sécurité Recette : 16,8 millions d'euros Date de création : 2006", "Taxe sur les titulaires d'ouvrages hydroélectriques concédés", "Péage plaisance", "Taxe sur le prix des entrées aux séances organisées par les exploitants d'établissements de spectacles cinématographiques", "Cotisations (normale et supplémentaire) des entreprises cinématographiques", "Redevance d'archéologie préventive Recette : 77 millions d'euros Date de création : 2001", "Redevances perçues pour la surveillance des établissements de jeux, hippodromes et cynodromes Recette : 0 millions d'euros Date de création : <1979", "Contribution des employeurs à l'association pour la gestion du régime d'assurance des créances des salariés", "Contribution annuelle au fonds de développement pour l'insertion professionnelle des handicapés", "Participation des entreprises de moins de 10 salariés au développement de la formation professionnelle continue", "PEFPC : Participation des entreprises de 10 salariés et plus au développement de la formation professionnelle continue", "Participation au financement de la formation des professions non salariées (hors artisanat, agriculture et pêche) Recette : 58 millions d'euros Date de création : 1991", "Participation au financement de la formation des travailleurs indépendants et des employeurs de la pêche maritime ou des cultures marines Recette : 0,4 millions d'euros Date de création : 1997", "Participation au financement de la formation des professions non salariées dans le domaine agricole Recette : 49 millions d'euros Date de création : 1991", "Participation au financement des congés individuels de formation des salariés sous contrats à durée déterminée CIF-CDD", "Contribution spéciale versée par les employeurs des étrangers sans autorisation de travail", "Prélèvement sur les contrats d'assurance-vie en déshérence", "Taxe dans le domaine funéraire Recette : 5 millions d'euros Date de création : 1806", "Taxe locale sur la publicité extérieure (TLPE) Recette : 153 millions d'euros Date de création : 2009", "Taxe sur les remontées mécaniques Recette : 54 millions d'euros Date de création : 1968", "Versement transport", "Taxe sur les activités commerciales non salariés à durée saisonnière    Date de création : 2000", "Taxe sur les activités commerciales saisonnières non salariées (TACDS)    Date de création : 2000", "Taxe sur les déchets réceptionnés dans une installation de stockage ou un incinérateur de déchets ménagers Recette : 18,9 millions d'euros Date de création : 2005", "Taxe additionnelle départementale à la taxe de séjour Recette : 9 millions d'euros Date de création : 1927", "Droits assimilés au droit d'octroi de mer sur les rhums et spiritueux à base d'alcool de cru Recette : 5,1 millions d'euros Date de création : 1963", "Impôt sur le revenu (IR) Recette : 74000 millions d'euros Date de création : 1916", "Taxe sur les métaux précieux, les bijoux, les objets d'art, de collection et d'antiquité Recette : 96,7 millions d'euros Date de création : 1976", "Impôt sur les revenus de capitaux mobiliers (IRCM)", "Impôt sur les sociétés (IS) Recette : 36200 millions d'euros Date de création : 1948", "Taxe sur les salaires", "Taxe annuelle sur les locaux à usage de bureaux, les locaux commerciaux, les locaux de stockage et les surfaces de stationnement perçue dans certains départements (IF-AUT-50)", "Taxe annuelle sur les logements vacants (IF-AUT-60)    Date de création : 1998", "contribution sur les revenus locatifs (CRL) Recette : 0,2 millions d'euros Date de création : 2000", "Taxe sur les excédents de provisions des entreprises d'assurances de dommages Recette : 114,6 millions d'euros Date de création : 1983", "Taxe sur les ordres annulés dans le cadre d'opération à haute fréquence Recette : 0,1 millions d'euros Date de création : 2012", "Taxe sur les contrats d'échange sur défaut d'un État Recette : 0,6 millions d'euros Date de création : 2012", "Taxe sur la valeur ajoutée (TVA) Recette : 141200 millions d'euros Date de création : 1954", "Taxe sur les services numériques Recette : 350 millions d'euros Date de création : 2019", "Taxe de solidarité sur les billets d'avion (dite taxe Unitaid ou taxe Chirac) Recette : 161,99 millions d'euros Date de création : 2006", "Taxe de l'aviation civile (TAC) Recette : 401 millions d'euros Date de création : 1999", "Taxe sur certaines dépenses de publicité", "Taxe sur le chiffre d'affaires des exploitants agricoles Recette : 138,2 millions d'euros Date de création : 1947", "Redevance sanitaire d'abattage Recette : 48 millions d'euros Date de création : 1989", "Redevance sanitaire de découpage Recette : 48,1 millions d'euros Date de création : 1989", "Redevance sanitaire de transformation des produits de la pêche et de l'aquaculture Recette : 0,3 millions d'euros Date de création : 1998", "Redevance sanitaire de transformation des produits de la pêche et de l'aquaculture Recette : 0,1 millions d'euros Date de création : 1998", "Redevance sanitaire pour le contrôle de certaines substances et de leurs résidus Recette : 0,8 millions d'euros Date de création : 1998", "Redevance pour l'agrément des établissements du secteur de l'alimentation animale Recette : 0 millions d'euros Date de création : 2009", "Taxe due par les concessionnaires d'autoroutes", "Contribution sur la cession à un service de télévision des droits de diffusion de manifestations ou de compétitions sportives Recette : 43 millions d'euros Date de création : 1999", "Prélèvements sur les jeux et paris Recette : 115,3 millions d'euros Date de création : 2010", "Fraction du Prélèvement sur les mises de jeux de cercle en ligne affectée aux communes dans le ressort territorial desquelles sont ouverts au public un ou plusieurs casinos", "Fraction du Prélèvement sur les paris hippiques affectée aux EPCI sur le territoire desquels sont ouverts au public un ou plusieurs hippodromes", "Droit de consommation sur les produits intermédiaires Recette : 104,7 millions d'euros Date de création : 1945", "Droits de consommation sur les alcools", "Droit de circulation sur les vins, cidres, poirés et hydromels Recette : 122,2 millions d'euros Date de création : 1945", "Droit sur les bières et les boissons non alcoolisées", "Droit de consommation sur les tabacs manufacturés", "Mutations à titre onéreux de fonds de commerce", "Droits de succession", "Droit fixe pour l'établissement d'un contrat de mariage Recette : 4,5 millions d'euros", "Fraction des droits de timbre sur les passeports sécurisés", "Droit de timbre sur les demandes de naturalisation, les demandes de réintégration dans la nationalité française et les déclarations d'acquisition de la nationalité en raison du mariage", "Impôt sur la fortune immobilière (PAT-IFI) Recette : 5043 millions d'euros Date de création : 1982", "Taxe spéciale sur les conventions d'assurances    Date de création : 1944", "Majoration de la taxe sur les assurances de protection juridique au profit Conseil national des barreaux", "Taxe sur les véhicules de sociétés (TVS)", "Taxe sur les véhicules de tourisme les plus polluants", "Malus (ou « écopastille »)", "Malus annuel", "Taxe foncière sur les propriétés bâties (IF-TFB) Recette : 27285 millions d'euros", "Taxe foncière sur les propriétés non bâties (IF-TFNB) Recette : 980 millions d'euros", "Taxe d'habitation (IF-TH) Recette : 19352 millions d'euros", "Taxe d'habitation sur les logements vacants    Date de création : 2007", "Cotisation foncière des entreprises (IF-CFE) Recette : 21872 millions d'euros", "Redevances communale et départementale des mines (TFP-MINES) Recette : 23,4 millions d'euros Date de création : 1919", "Imposition forfaitaire sur les pylônes (TFP-PYL) Recette : 343,4 millions d'euros Date de création : 1980", "Taxe sur les éoliennes maritimes (TFP-TEM) Recette : 0 millions d'euros Date de création : 2005", "Imposition forfaitaire sur les éoliennes et les hydroliennes Recette : 44,9 millions d'euros Date de création : 2010", "Imposition forfaitaire sur les centrales de production d'énergie électrique d'origine photovoltaïque ou hydraulique Recette : 75,9 millions d'euros Date de création : 2010", "Imposition forfaitaire sur les réseaux de gaz naturel et canalisations d'hydrocarbures Recette : 38,5 millions d'euros Date de création : 2010", "Redevances sur la production d'électricité au moyen de la géothermie Recette : 0 millions d'euros Date de création : 2017", "Taxe additionnelle à la taxe foncière sur les propriétés non bâties (IF-AUT-80)", "Taxe d'enlèvement des ordures ménagères (IF-AUT-90)) Recette : 6087 millions d'euros", "Taxe de balayage Recette : 108,9 millions d'euros Date de création : 1873", "Taxe forfaitaire sur la cession à titre onéreux des terrains nus qui ont été rendus constructibles du fait de leur classement Recette : 54 millions d'euros Date de création : 2006", "Taxe pour la gestion des milieux aquatiques et la prévention des inondations", "Taxe sur les friches commerciales (IF-AUT-110)    Date de création : 2006", "Impôt sur les cercles et maisons de jeux Recette : 34,5 millions d'euros Date de création : 1941", "Surtaxe sur les eaux minérales", "Taxe perçue au profit des communes de Saint-Martin et Saint-Barthélemy", "Taxe sur l'exploration d'hydrocarbures (TFP-TEH) Recette : 0,8 millions d'euros Date de création : 2017", "Taxe de publicité foncière", "Droits départementaux d'enregistrement sur les mutations à titre onéreux d'immeubles", "Taxe départementale additionnelle aux droits d'enregistrement ou à la taxe de publicité foncière exigible sur les mutations à titre onéreux Recette : 99,8 millions d'euros Date de création : 1798", "Taxe d'apprentissage Recette : 2000 millions d'euros Date de création : 1925", "Imposition forfaitaire sur le matériel roulant circulant sur le réseau de transport ferroviaire et guidé géré par la RATP -IFER-STIF RATP", "Taxe annuelle sur les surfaces de stationnement    Date de création : 2015", "Taxe additionnelle spéciale annuelle au profit de la région Île-de-France (TASA)    Date de création : 2015", "Taxe perçue pour la région de Guyane (TFP-GUF) Recette : 0,4 millions d'euros Date de création : 2008", "Taxe sur les permis de conduire", "Taxe régionale sur les certificats d'immatriculation des véhicules", "Taxe due par les entreprises de transport public aérien et maritime (Corse) Recette : 47,4 millions d'euros Date de création : 1991", "Taxe pour frais de chambres de commerce et d'industrie (IF-AUT-10)", "Contribution sociale généralisée (CSG) Recette : 99000 millions d'euros Date de création : 1991", "Prélèvement social sur les revenus du patrimoine et les produits de placements", "Contribution pour le remboursement de la dette sociale (CRDS) Recette : 6150 millions d'euros Date de création : 1996", "Prélèvement de solidarité de 2 % sur les revenus du patrimoine et les produits de placements", "Taxe pour frais de chambres de métiers et de l'artisanat (IF-AUT-20)", "Contribution au fonds d'assurance formation des chefs d'entreprise inscrite au répertoire des métiers Recette : 58 millions d'euros Date de création : 1982", "Taxe pour frais de chambres d'agriculture (IF-AUT-30)", "Taxe sur la cession à titre onéreux de terrains nus devenus constructibles; perçue au profit de l'agence de services et de paiement Recette : 11 millions d'euros Date de création : 2010", "Contribution à l'audiovisuel public due par les professionnels (TFP-CAP) Recette : 3500 millions d'euros Date de création : 1933", "Taxe spéciale d'équipement Recette : 350 millions d'euros Date de création : 1991", "Taxe spéciale d'équipement au profit de l'EPF de Normandie Recette : 13 millions d'euros Date de création : 1968", "Redevance sur les paris hippiques en ligne perçue au profit des sociétés de courses Recette : 61 millions d'euros Date de création : 2014", "Taxe sur les nuisances sonores aériennes Recette : 57 millions d'euros Date de création : 1992", "Contribution des autoentrepreneurs au financement des actions de formation des chambres de métiers et d'artisanat Recette : 2 millions d'euros Date de création : 2010", "Contribution des autoentrepreneurs au fonds d'assurance formation des chefs d'entreprise artisanale Recette : 3 millions d'euros Date de création : 2010", "Taxe sur la diffusion en vidéo physique et en ligne de contenus audiovisuels Recette : 30,95 millions d'euros Date de création : 1993", "Contribution supplémentaire à l'apprentissage - versements aux CFA", "Taxe pour le développement de la formation professionnelle dans les métiers de la réparation automobile, du cycle et du motocycle Recette : 32 millions d'euros Date de création : 1968", "Prélèvements sur les jeux de loterie et les paris sportifs perçus au profit du Centre national pour le développement du sport Recette : 35,9 millions d'euros Date de création : 2010", "Taxe spéciale d'équipement au profit de l'EPF de Guyane et de Mayotte Recette : 2 millions d'euros Date de création : 1994", "Taxe spéciale d'équipement au profit de la Société du Grand Paris Recette : 117 millions d'euros Date de création : 2010", "Taxe spéciale d'équipement au profit de l'EPF de Lorraine Recette : 23 millions d'euros Date de création : 1973", "Taxe spéciale d'équipement au profit de l'EPF de PACA Recette : 50 millions d'euros Date de création : 2001", "Taxe sur les boissons prémix Recette : 2,3 millions d'euros Date de création : 1996", "Contribution perçue sur les boissons et préparations liquides destinées à la consommation humaine contenant des édulcorants de synthèse et ne contenant pas de sucres ajoutés Recette : 58,4 millions d'euros Date de création : 2011", "Droit de timbre pour la délivrance du permis de conduire en cas de perte ou de vol", "Taxes à percevoir pour l'alimentation du fonds commun des accidents du travail agricole Recette : 17,3 millions d'euros Date de création : 1957", "Fraction des droits de timbre sur les cartes nationales d'identité", "Taxe pour la gestion des certificats d'immatriculation des véhicules Recette : 42 millions d'euros Date de création : 2009", "Contributions additionnelles aux primes ou cotisations afférentes à certaines conventions d'assurance", "Contribution au Fonds de prévention des risques naturels majeurs (FPRNM, dit « fonds Barnier »)    Date de création : 1995", "Droits perçus au profit de la Caisse nationale de l'assurance maladie des travailleurs salariés (CNAMTS) en matière de produits de santé, taxe annuelle due par les laboratoires de biologie médicale", "Taxe destinée à financer le développement de la formation professionnelle dans les transports routiers Recette : 65 millions d'euros Date de création : 1968", "Droit d'examen du permis de chasse", "Droits affectés au fonds d'indemnisation de la profession d'avoués près les cours d'appel Recette : 16 millions d'euros Date de création : 2011", "Imposition forfaitaire sur les entreprises de réseaux (TFP-IFER) Recette : 1337 millions d'euros Date de création : 2010", "Taxe communale sur la consommation finale d'électricité (TCFE) Recette : 61,9 millions d'euros Date de création : 2010", "Taxe départementale des espaces naturels sensibles (TDENS)", "Redevance d'exploitation de substances non énergétiques sur le plateau continental ou dans la zone économique exclusive", "Redevance due par les titulaires de titres d'exploitation de mines d'hydrocarbures liquides ou gazeux Recette : 3,1 millions d'euros Date de création : 1956", "Redevance due par les titulaires de titres d'exploitation de mines d'hydrocarbures liquides ou gazeux au large de Saint-Pierre-et-Miquelon    Date de création : 1999", "Contribution pour frais de contrôle ACPR", "Droits et contributions pour frais de contrôle", "Redevance pour contrôle vétérinaire à l'expédition    Date de création : <2000", "Redevance relative aux contrôles renforcés à l'importation des denrées alimentaires d'origine non animale Recette : 1,1 millions d'euros Date de création : 1998", "Contribution des exploitants agricoles et des conchyliculteurs au Fonds national de gestion des risques en agriculture Recette : 102 millions d'euros Date de création : 1993", "Droit sur les produits bénéficiant d'une appellation d’origine, d'une indication géographique ou d'un label rouge Recette : 7 millions d'euros Date de création : 1935", "Participation au financement de la formation des professions non salariées dans le domaine agricole", "Taxe additionnelle à la taxe sur les installations nucléaires de base - stockage Recette : 2,4 millions d'euros Date de création : 2009", "Taxe additionnelle à la taxe sur les installations nucléaires de base - Diffusion technologique Recette : 20 millions d'euros Date de création : 2006", "Taxe additionnelle à la taxe sur les installations nucléaires de base - Accompagnement Recette : 39 millions d'euros Date de création : 2006", "Taxe pour le développement des industries de l'ameublement ainsi que les industries du bois Recette : 14,9 millions d'euros Date de création : 1971", "Taxe pour le développement des industries du cuir, de la maroquinerie, de la ganterie et de la chaussure Recette : 12,5 millions d'euros Date de création : 1978", "Taxe pour le développement des industries de l'habillement Recette : 10 millions d'euros Date de création : 1963", "Taxe pour le développement des industries de la mécanique, de la construction métallique, des matériels etc. Recette : 70,2 millions d'euros Date de création : 1961", "Taxe pour le développement des industries de l'horlogerie, bijouterie, joaillerie et orfèvrerie ainsi que des arts de la table (taxe HBJOAT) Recette : 13,2 millions d'euros Date de création : 1963", "Taxe pour le développement des industries des matériaux de construction regroupant les industries du béton, de la terre cuite et des roches ornementales et de construction Recette : 15,9 millions d'euros Date de création : 1957", "Taxe pour le développement de l'industrie de la conservation des produits agricoles Recette : 2,6 millions d'euros Date de création : 1950", "Taxe sur les spectacles de variétés Recette : 24 millions d'euros Date de création : 1977", "Taxe sur les spectacles d'art dramatique, lyrique et chorégraphique Recette : 5,1 millions d'euros Date de création : 1964", "Redevance pour frais d'envoi des certificats d'immatriculation des véhicules", "Fraction affectée du produit du relèvement du tarif de taxe intérieure de consommation sur les produits énergétiques (TICPE) sur le carburant gazole", "Taxe sur les hydrofluorocarbures    Date de création : 2018", "Contribution annuelle au profit de l'Institut de radioprotection et de sûreté nucléaire (IRSN) Recette : 48 millions d'euros Date de création : 2010", "Taxe sur les transactions financières (TTF)", "Participation pour le Financement de l'Assainissement Collectif (PFAC)", "Contribution spéciale pour la gestion des déchets radioactifs - Conception", "Participation des concessionnaires de la liaison fixe Trans-Manche au fonctionnement de la commission intergouvernementale et du comité de sécurité chargés de superviser la construction et l'exploitation de l'ouvrage Recette : 2,5 millions d'euros Date de création : 1986", "Redevance proportionnelle sur l'énergie hydraulique", "Taxe pour frais de chambre de métiers de Moselle Recette : 7 millions d'euros Date de création : 1919", "Taxe pour frais de chambre de métiers d'Alsace Recette : 9 millions d'euros Date de création : 1919", "Prélèvements sur les bénéfices tirés de la construction immobilière Recette : 0,4 millions d'euros Date de création : 1963", "Participation des employeurs au financement de la formation professionnelle continue, versée à l'État", "Taxe sur les surfaces commerciales (TFP-TSC) Recette : 609 millions d'euros", "Taxe additionnelle à la taxe sur les surfaces commerciales (TFP-TASC)", "Taxe exceptionnelle sur la réserve de capitalisation des entreprises d'assurance (TFP-ASSUR)", "Taxe au profit du fonds de soutien aux collectivités territoriales ayant contracté des produits structurés dits \"emprunts toxiques\" (TFP-TFSCT)", "Cotisation obligatoire", "Cotisation obligatoire", "Taxe professionnelle de la Poste et de France Telecom", "Fraction du produit des successions en déshérence", "Droit d'octroi de mer et droit d'octroi de mer régional", "Contribution tarifaire d'acheminement (CTA)    Date de création : 2004", "Contributions versées par la SNCF au titre des frais de surveillance et de contrôle des chemins de fer    Date de création : 1981", "Redevance versée par Réseau ferré de France au titre des frais de surveillance et de contrôle    Date de création : 1981", "Contributions des employeurs de main d'œuvre étrangère pour l'OMI", "Contribution des employeurs publics au fonds pour l'insertion des personnes handicapées dans la fonction publique", "Cotisation au profit des caisses d'assurances d'accidents agricoles d'Alsace-Moselle Recette : 12 millions d'euros Date de création : 1912", "Droits d'apport des sociétés", "Droits de donations", "Mutations à titre onéreux de créances, rentes, prix d'offices", "Contributions au Fonds national de l'emploi (FNE)", "Retenue à la source sur certains bénéfices non commerciaux et de l'impôt sur le revenu", "Contribution des institutions financières", "Cotisations aux fonds de garantie des salaires (AGS et AGCC)", "Redevance d'usage des fréquences radioélectriques", "Redevances lors du lancement de certains matériels aéronautiques", "Taxe grossiste répartiteurs", "Taxe sur les stations et liaisons radio privées", "Taxe additionnelle aux droits de mutation", "Impôt sur les spectacles, jeux et divertissements", "Participation dépassement du COS", "Taxe locale d'équipement", "Taxe complémentaire à la TLE (IdF)", "Taxe de séjour", "Taxe sur les tabacs (Corse)", "Octroi de mer", "Taxe sur le ski de fond Recette : 10 millions d'euros Date de création : 1985", "Contribution annuelle des distributeurs d'énergie", "Fonds d'amortissement des charges d'électrification (FACÉ)", "Prélèvement sur entreprises pétrolières", "Taxe sur les fournitures d'électricité", "Droits de consommation sur les alcools (Corse)", "Taxe d'assainissement (Agence de l'Eau)", "Taxe sur les rhums", "Taxe sur les carburants (DOM)", "Taxe sur les syndicats d'énergie", "Redevance pour droit de construire (EPAD)", "Taxe et prélèvement sur les sommes encaissées par les sociétés de télévision au titre de la redevance, de la diffusion des messages publicitaires et des abonnements Recette : 8 millions d'euros Date de création : 1946", "Mutation à titre onéreux d'immeubles et droits immobiliers (Droits de mutation)", "Mutations de jouissance (baux)", "Fraction des Prélèvements sociaux sur les jeux prévus aux Art L 137-20 à L 137-22 Code de la sécurité sociale", "Prélèvement sur la participation des employeurs à l'effort de construction", "Cotisation obligatoire", "Taxe fixe due à chaque délivrance de CI (AIS-MOB-10-20-20)", "Taxe régionale due au titre de la délivrance de CI consécutive d'un changement de propriétaire", "Taxe sur les véhicules de transport due au titre de la délivrance de CI consécutive à un changement de propriétaire", "Taxe sur les émissions de dioxyde de carbone", "Taxe sur la masse en ordre de marche", "Taxe annuelle sur les émissions de dioxyde de carbone", "Taxe annuelle sur les émissions de polluants atmosphérique", "Taxe sur l'affectation des véhicules lourds de transport de marchandises (AIS-MOB-10-30-30)", "Taxe sur la distance parcourue sur le réseau autoroutier concédé (AIS-MOB-10-40)", "Taxes sur le transport aérien (AIS-MOB-20)", "Taxes sur les navigations maritimes et fluviales (AIS-MOB-30)", "Taxes sur le transport guidé (AIS-MOB-40)", "Taxe sur l'exploitation des infrastructures de transport de longue distance (AIS-MOB-50)", "Taxe sur les services de communications électroniques (AIS-CCN-30-10)", "Taxe sur les services de télévision (AIS-CCN-30-20)", "Taxe sur les services d'accès à des contenus audiovisuels à la demande (AIS-CCN-30-30)", "Taxe sur la mise en relation par voie électronique en vue de fournir certaines prestations de transport (AIS-CCN-30-40) Recette : 2 millions d'euros Date de création : 2021", "Taxe sur certains services numériques (AIS-CCN-30-50)", "Taxe sur les locations en France de phonogrammes musicaux et de vidéomusiques destinés à l’usage privé du public dans le cadre d’une mise à disposition à la demande sur les réseaux en ligne (AIS-CCN-30-60) Recette : 9,3 millions d'euros Date de création : 2023", "Cotisation sur la valeur ajoutée des entreprises (CVAE)", "Prélèvements au profit de l'État (IF-AUT-40)", "Taxe spéciale complémentaire au profit de la Société du Grand Projet du Sud-Ouest (IF-AUT-150)", "Taxe spéciale d'équipement au profit des établissements publics fonciers locaux et de l'office foncier de Corse", "Taxe spéciale d'équipement au profit des établissements publics fonciers de l’État", "Taxe spéciale d'équipement au profit des établissements publics fonciers et d'aménagement de la Guyane et de Mayotte", "Taxe spéciale d'équipement au profit de l'agence pour la mise en valeur des espaces urbains de la zone dite des cinquante pas géométriques en Guadeloupe", "Taxe spéciale d'équipement au profit de l'agence pour la mise en valeur des espaces urbains de la zone dite des cinquante pas géométriques en Martinique.", "Taxe spéciale d'équipement au profit de l'établissement public Société des grands projets", "Taxe spéciale d'équipement au profit de l'établissement public local Société du Grand Projet du Sud-Ouest", "Taxe additionnelle spéciale annuelle perçue au profit de la région Île-de-France (IF-AUT-130)", "Taxe annuelle sur les surfaces de stationnement perçue en Île-de-France (IF-AUT-140)", "Taxe sur la valeur vénale des immeubles des entités juridiques (PAT-TPC)", "Contribution exceptionnelle sur les hauts revenus (IR-CHR)", "Taxe additionnelle à l’accise sur les tabacs    Date de création : 2025", "Participation des employeurs agricoles à l'effort de construction (TPS-PEEC-60)    Date de création : 2006", "Cotisation additionnelle assise sur le montant total des honoraires facturés par les commissaires aux comptes lorsqu'ils certifient les comptes ou les informations en matière de durabilité des entités d'intérêt public    Date de création : 2017", "Contribution forfaitaire des commissaires aux comptes exerçant dans les pays tiers    Date de création : 2017", "Cotisation assise sur le montant total des honoraires facturés par les organismes tiers indépendants lorsqu'ils certifient les informations en matière de durabilité    Date de création : 2017", "Cotisation additionnelle assise sur le montant total des honoraires facturés par les organismes tiers indépendants lorsqu'ils certifient les informations en matière de durabilité des entités d'intérêt public    Date de création : 2017", "Eco-contribution sur la taxe de solidarité sur les billets d'avion", "Taxe d'aéroport", "Taxe d'embarquement sur les passagers dans les territoires d'outre-mer", "Taxe d'atterrissage", "CSG Recette : 147500 millions d'euros", "CRDS Recette : 8853 millions d'euros Date de création : 1996", "Contribution solidarité autonomie", "Assurance maladie - maternité - invalidité - décès", "Assurance vieillesse plafonnée", "Assurance vieillesse déplafonnée", "Allocations familiales Recette : 35600 millions d'euros", "Accidents du travail Recette : 16000 millions d'euros", "Aide au logement entreprise de moins de 50 salariés (FNAL)", "Supplément entreprise de 50 salariés et plus (FNAL)", "Cotisation chômage", "Fonds national de garantie des salaires (AGS)", "Retraite complémentaire CEG tranche 1", "Retraite complémentaire CEG tranche 2", "Retraite complémentaire APEC (cadres seulement)", "Retraite complémentaire Contribution patronale de prévoyance (forfait social) entreprises de 11 à 49 salariés", "Retraite complémentaire Contribution patronale de prévoyance (forfait social) entreprises de 50 salariés et plus", "Retraite complémentaire Cotisations de base", "Retraite complémentaire tranche 1", "Retraite complémentaire tranche 2", "Retraite complémentaire CET : T1 + T2 si rémunération supérieure au plafond de la Sécurité sociale (3.666 €)", "assurance décès cadre (adhésion obligatoire pour les cadres quel que soit le secteur d'activité)", "Contribution formation professionnelle Entreprise de moins de 11 salariés", "Contribution formation professionnelle Entreprise de 11 salariés ou plus", "Contribution formation professionnelle Entreprise avec CDD (CPF-CDD)", "Taxe d'apprentissage", "Taxe sur les salaires (employeur non assujetti à la TVA)", "Versement mobilité (transport) entreprises de 11 salariés et plus", "Participation à l'effort de construction (entreprises de 50 salariés et plus)", "Contribution au Dialogue social"];

const canvas = document.getElementById('wheel');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;
const CX = W/2, CY = H/2;
const R = Math.min(W,H)*0.48;

// --- Canvas hors écran ---
const offscreen = document.createElement('canvas');
offscreen.width = W;
offscreen.height = H;
const offctx = offscreen.getContext('2d');

let angle = -Math.PI/2;
let spinning = false;
let colors = [];
let lastPointerIndex = null;
let lastPingTime = 0;

/* -------- Couleurs alternées -------- */
function buildColors(n){
  const palette = [
    "#f87171", // rouge clair
    "#60a5fa", // bleu clair
    "#facc15", // jaune
    "#34d399", // vert
    "#a78bfa", // violet
    "#fb923c"  // orange
  ];
  const arr = [];
  for(let i=0;i<n;i++) arr.push(palette[i % palette.length]);
  return arr;
}

/* -------- Font -------- */
function computeFont(n){
  if(n>340) return 7;
  if(n>260) return 8;
  if(n>200) return 9;
  if(n>140) return 10;
  return 11;
}

/* -------- Roue statique -------- */
function buildStaticWheel(){
  offctx.clearRect(0,0,W,H);
  const n = ENTRIES.length;
  if(n===0) return;
  const step = (Math.PI*2)/n;
  offctx.save();
  offctx.translate(CX, CY);

  for(let i=0;i<n;i++){
    const start=i*step,end=start+step;
    offctx.beginPath();
    offctx.moveTo(0,0);
    offctx.arc(0,0,R,start,end);
    offctx.closePath();
    offctx.fillStyle=colors[i%colors.length];
    offctx.fill();
    if(n<=600){
      offctx.strokeStyle="rgba(0,0,0,.18)";
      offctx.lineWidth=0.5;
      offctx.stroke();
    }
  }

  const fontPx=computeFont(n);
  offctx.fillStyle="#111";
  offctx.textAlign="center";
  offctx.textBaseline="middle";
  offctx.font=`${fontPx}px Helvetica, Arial, sans-serif`;
  const labelRadius=R*0.78;
  for(let i=0;i<n;i++){
    const mid=(i+0.5)*step;
    const x=Math.cos(mid)*labelRadius;
    const y=Math.sin(mid)*labelRadius;
    const raw=String(ENTRIES[i]??"");
    const label=raw.length>36?raw.slice(0,36)+"…":raw;
    offctx.save();
    offctx.translate(x,y);
    offctx.rotate(mid);
    offctx.fillText(label,0,0);
    offctx.restore();
  }

  offctx.beginPath();
  offctx.arc(0,0,R*0.12,0,Math.PI*2);
  offctx.fillStyle="#fff";
  offctx.strokeStyle="#e5e7eb";
  offctx.lineWidth=1;
  offctx.fill();
  offctx.stroke();
  offctx.restore();
}

/* -------- Affichage -------- */
function drawWheel(a){
  ctx.clearRect(0,0,W,H);
  ctx.save();
  ctx.translate(CX,CY);
  ctx.rotate(a);
  ctx.drawImage(offscreen,-CX,-CY);
  ctx.restore();
}

/* -------- Sélection -------- */
function getSelectedIndex(a){
  const n=ENTRIES.length;
  const step=(Math.PI*2)/n;
  let theta=(-Math.PI/2 - a) % (Math.PI*2);
  if(theta<0) theta += Math.PI*2;
  return Math.floor(theta/step);
}

/* -------- Easing -------- */
function easeInOutCubic(x){return x<0.5?4*x*x*x:1-Math.pow(-2*x+2,3)/2;}

/* -------- Overlay -------- */
const overlay=document.getElementById('overlay');
const overlayText=document.getElementById('overlayText');
const overlayClose=document.getElementById('overlayClose');
function showOverlay(text){
  overlayText.textContent=text;
  overlay.style.display='flex';
}
overlayClose.addEventListener('click',()=>overlay.style.display='none');

/* -------- Nouveaux sons -------- */
let spinSound = new Audio("wheel-spin.mp3");
let coinSound = new Audio("coin.mp3");
spinSound.loop = true;
spinSound.volume = 0.7;
coinSound.volume = 0.9;

function playSpinSound(){
  spinSound.currentTime = 0;
  spinSound.play().catch(()=>{});
}
function stopSpinSound(){
  spinSound.pause();
  spinSound.currentTime = 0;
}
function playCoinSound(){
  coinSound.currentTime = 0;
  coinSound.play().catch(()=>{});
}

/* -------- Animation du spin -------- */
function spin(){
  if(spinning) return;
  if(ENTRIES.length===0){alert("Plus aucun élément à tirer.");return;}
  spinning=true;
  const btn=document.getElementById('spinBtn');
  btn.disabled=true;

  const startAngle = angle; // point de départ réel
  const duration=4000;
  const totalTurns=4+Math.random()*2;
  const finalOffset=Math.random()*Math.PI*2;
  const totalAngle=totalTurns*Math.PI*2+finalOffset;
  const start=performance.now();
  lastPointerIndex=getSelectedIndex(startAngle);
  lastPingTime=start;

  playSpinSound(); // 🔊 démarre le son de rotation

  function animate(ts){
    const t=Math.min(1,(ts-start)/duration);
    const eased=easeInOutCubic(t);
    angle=startAngle + eased*totalAngle;
    drawWheel(angle);

    const idx=getSelectedIndex(angle);
    if(idx!==lastPointerIndex && (ts-lastPingTime)>25){
      lastPointerIndex=idx;
      lastPingTime=ts;
    }

    if(t<1){
      requestAnimationFrame(animate);
    }else{
      stopSpinSound(); // 🔇 stop son roue
      const winnerIdx=getSelectedIndex(angle);
      const chosen=ENTRIES[winnerIdx];
      playCoinSound(); // 🔊 son pièce
      showOverlay("🎯 "+chosen);

      // normalise l'angle
      angle=((angle % (Math.PI*2)) + Math.PI*2) % (Math.PI*2);

      // supprime le gagnant
      ENTRIES.splice(winnerIdx,1);
      colors=buildColors(ENTRIES.length);
      buildStaticWheel();
      document.getElementById('countInfo').textContent=ENTRIES.length+" éléments restants";
      drawWheel(angle);
      btn.disabled=ENTRIES.length===0;
      spinning=false;
    }
  }
  requestAnimationFrame(animate);
}

/* -------- Init -------- */
document.getElementById('spinBtn').addEventListener('click',spin);
colors=buildColors(ENTRIES.length);
buildStaticWheel();
document.getElementById('countInfo').textContent=ENTRIES.length+" éléments";
drawWheel(angle);
</script>
</body>
</html>
