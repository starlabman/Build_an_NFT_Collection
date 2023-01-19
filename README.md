Build an NFT Collection
Now its time for you to launch your own NFT collection - Crypto Devs.

Image

Requirements
Il ne devrait exister que 20 NFT Crypto Dev et chacun d'eux devrait être unique.
Les utilisateurs ne devraient pouvoir frapper que 1 NFT avec une seule transaction.
Les utilisateurs blanchis devraient avoir une période de prévente de 5 minutes avant la vente réelle où ils sont garantis 1 NFT par transaction.
Il devrait y avoir un site Web pour votre collection NFT.
Commençons à construire 🚀

Prérequis
Vous auriez dû terminer le Whitelist dApp tutoriel de tout à l'heure.
Théorie
Qu'est-ce qu'un jeton non fongible? Fongible signifie être le même ou interchangeable, par exemple. Eth est fongible. Dans cet esprit, les NFT sont uniques; chacun est différent. Chaque jeton a des caractéristiques et des valeurs uniques. Ils se distinguent tous les uns des autres et ne sont pas interchangeables, par exemple l'art unique

Qu'est-ce que l'ERC-721? ERC-721 est une norme ouverte qui décrit comment construire des jetons non fongibles sur des blockchains compatibles EVM ( Ethereum Virtual Machine ); il s'agit d'une interface standard pour les jetons non fongibles; il a un ensemble de règles qui facilitent le travail avec les NFT. Avant d'aller de l'avant, jetez un œil à toutes les fonctions prises en charge par ERC721

Construire
Vous préférez une vidéo?
Si vous préférez apprendre d'une vidéo, nous avons un enregistrement disponible de ce tutoriel sur notre YouTube. Regardez la vidéo en cliquant sur la capture d'écran ci-dessous, ou allez-y et lisez le tutoriel!ImageImage

Contrat intelligent
Nous utiliserons Ownable.sol d'Openzeppelin qui vous aide à gérer le Ownership d'un contrat

Par défaut, le propriétaire d'un contrat propriétaire est le compte qui l'a déployé, ce qui est généralement exactement ce que vous voulez.
Propriétaire vous permet également:
transferOwnership du compte propriétaire à un nouveau, et
renoncez à la propriété pour que le propriétaire renonce à ce privilège administratif, un schéma commun après une première étape avec une administration centralisée est terminé.
Nous utiliserons également une extension d'ERC721 connue sous le nom de ERC721 énumérable

ERC721 Enumerable vous aide à suivre tous les jetons du contrat ainsi que les jetons détenus par une adresse pour un contrat donné.
Veuillez jeter un œil au fonctions il met en œuvre avant d'avancer
Pour construire le contrat intelligent que nous utiliserions Hardhat. Hardhat est un environnement et un cadre de développement Ethereum conçus pour le développement complet de la pile dans Solidity. En termes simples, vous pouvez rédiger votre contrat intelligent, les déployer, exécuter des tests et déboguer votre code.

To setup a Hardhat project, Open up a terminal and execute these commands

mkdir NFT-Collection
cd NFT-Collection
mkdir hardhat-tutorial
cd hardhat-tutorial
npm init --yes
npm install --save-dev hardhat
If you are a Windows user, you'll have to add one more dependency. so in the terminal, add the following command :

npm install --save-dev @nomicfoundation/hardhat-toolbox
In the same directory where you installed Hardhat run:

npx hardhat
Make sure you select Create a Javascript Project and then follow the steps in the terminal to complete your Hardhat setup.

Dans le même terminal, installez maintenant @openzeppelin/contracts comme nous importerions ERC721 d'Openzeppelin Contrat énumérable dans notre CryptoDevs contrat.

npminstaller @openzeppelin / contrats 
Nous devrons appeler le Whitelist Contract que vous avez déployé pour votre niveau précédent pour vérifier les adresses qui ont été blanchies et leur donner un accès prévente. Comme nous n'avons qu'à appeler mapping(address => bool) public whitelistedAddresses; Nous pouvons créer une interface pour Whitelist contract avec une fonction uniquement pour ce mappage, de cette façon, nous enregistrerions gas car nous n'aurions pas besoin d'hériter et de déployer l'ensemble Whitelist Contract mais seulement une partie de celui-ci.

Créer un nouveau fichier à l'intérieur du contracts répertoire et appelez-le IWhitelist.sol.

Remarque: les fichiers de solidité qui incluent uniquement des interfaces sont souvent préfixés avec I pour signifier qu'ils ne sont qu'une interface.

// SPDX-License-Identifier: MIT
solidité pragma ^0,8.4;

interface IWhitelist {
    fonction adresses blanchissées(adresse) retours de vue externe (bool);
}
Créons maintenant un nouveau fichier à l'intérieur du contracts répertoire et appelez-le CryptoDevs.sol

// SPDX-License-Identifier: MIT
solidité pragma ^0,8.4;

importation"@openzeppelin / contrats / token / ERC721 / extensions / ERC721Enumerable.sol"; 
importation"@openzeppelin / contrats / access / Ownable.sol"; 
importation"./IWhitelist.sol"; 

contract CryptoDevs is ERC721Enumerable, Ownable {
    /**
      * @dev _baseTokenURI for computing {tokenURI}. If set, the resulting URI for each
      * token will be the concatenation of the `baseURI` and the `tokenId`.
      */
    string _baseTokenURI;

    //  _price is the price of one Crypto Dev NFT
    uint256 public _price = 0.01 ether;

    // _paused is used to pause the contract in case of an emergency
    bool public _paused;

    // max number of CryptoDevs
    uint256 public maxTokenIds = 20;

    // total number of tokenIds minted
    uint256 public tokenIds;

    // Whitelist contract instance
    IWhitelist whitelist;

    // boolean to keep track of whether presale started or not
    bool public presaleStarted;

    // timestamp for when presale would end
    uint256 public presaleEnded;

    modifier onlyWhenNotPaused {
        require(!_paused, "Contract currently paused");
        _;
    }

    /**
      * @dev ERC721 constructor takes in a `name` and a `symbol` to the token collection.
      * name in our case is `Crypto Devs` and symbol is `CD`.
      * Constructor for Crypto Devs takes in the baseURI to set _baseTokenURI for the collection.
      * It also initializes an instance of whitelist interface.
      */
    constructor (string memory baseURI, address whitelistContract) ERC721("Crypto Devs", "CD") {
        _baseTokenURI = baseURI;
        whitelist = IWhitelist(whitelistContract);
    }

    /**
    * @dev startPresale starts a presale for the whitelisted addresses
      */
    function startPresale() public onlyOwner {
        presaleStarted = true;
        // Set presaleEnded time as current timestamp + 5 minutes
        // Solidity a une syntaxe cool pour les horodatages ( secondes, minutes, heures, jours, années )
        presaleEnded = bloquer.horodatage +5 minutes; 
    }

    /**
      * @dev presaleMint permet à un utilisateur de frapper un NFT par transaction pendant la prévente.
      */
    fonction presaleMint() public payable seulement quand {
        exiger(presaleDémarré && bloquer.horodatage < presaleEnded,"La prévente ne fonctionne pas"); 
        exiger(whitelist.adresses blanchissées(msg.expéditeur),"Vous n'êtes pas blanchi"); 
        exiger(tokenIds < maxTokenIds,"A dépassé l'alimentation maximale des Crypto Devs"); 
        exiger(msg.valeur > = _prix,"L'éther envoyé n'est pas correct"); 
        tokenIds + =1; 
        //_safeMint est une version plus sûre de la fonction _mint car elle garantit que
        // si l'adresse indiquée est un contrat, il sait comment traiter les jetons ERC721
        // If the address being minted to is not a contract, it works the same way as _mint
        _safeMint(msg.sender, tokenIds);
    }

    /**
    * @dev mint allows a user to mint 1 NFT per transaction after the presale has ended.
    */
    function mint() public payable onlyWhenNotPaused {
        require(presaleStarted && block.timestamp >=  presaleEnded, "Presale has not ended yet");
        require(tokenIds < maxTokenIds, "Exceed maximum Crypto Devs supply");
        require(msg.value >= _price, "Ether sent is not correct");
        tokenIds += 1;
        _safeMint(msg.sender, tokenIds);
    }

    /**
    * @dev _baseURI overides the Openzeppelin's ERC721 implementation which by default
    * returned an empty string for the baseURI
    */
    fonction _baseURI() retour de remplacement virtuel de vue interne (chaîne mémoire){ 
        retour _baseTokenURI;
    }

    /**
    * @dev setPaused fait suspendre ou non le contrat
      */
    fonction setPaused(bool val) public onlyOwner {
        _paused = val;
    }

    /**
    * @dev repli envoie tout l'éther dans le contrat
    * au propriétaire du contrat
      */
    fonction retirer() public onlyOwner  {
        adresse _propriétaire =propriétaire(); 
        montant uint256 =adresse(cette).équilibre; 
        (bool envoyé,)=  _propriétaire.appel{valeur: montant}("");  
        exiger(envoyé,"N'a pas envoyé Ether"); 
    }

      // Fonction pour recevoir Ether. msg.data doit être vide
    recevoir() à payer externe {}

    // La fonction de secours est appelée lorsque msg.data n'est pas vide
    repli() à payer externe {}
}
Maintenant installons dotenv package pour pouvoir importer le fichier env et l'utiliser dans notre configuration. Ouvrez un terminal pointant vers hardhat-tutorial répertoire et exécutez cette commande

npminstaller dotenv 
Maintenant, créez un .env fichier dans le hardhat-tutorial dossier et ajouter les lignes suivantes. Suivez les instructions ci-dessous.

Aller à Quicknode et inscrivez-vous pour un compte. Si vous avez déjà un compte, connectez-vous. Quicknode est un fournisseur de nœuds qui vous permet de vous connecter à différentes chaînes de blocs. Nous l'utiliserons pour déployer notre contrat via Hardhat. Après avoir créé un compte, Create an endpoint sur Quicknode, sélectionnez Ethereum, puis sélectionnez le Goerli réseau. Cliquez sur Continue en bas à droite puis cliquez sur Create Endpoint. Copiez le lien qui vous est donné dans HTTP Provider et l'ajouter au .env fichier ci-dessous pour QUICKNODE_HTTP_URL.

REMARQUE: Si vous avez précédemment configuré un point final Goerli sur Quicknode pendant la piste Freshman, vous pouvez utiliser la même URL qu'auparavant. Pas besoin de le supprimer et d'en créer un nouveau.

To get your private key, you need to export it from Metamask. Open Metamask, click on the three dots, click on Account Details and then Export Private Key. MAKE SURE YOU ARE USING A TEST ACCOUNT THAT DOES NOT HAVE MAINNET FUNDS FOR THIS. Add this Private Key below in your .env file for PRIVATE_KEY variable.

QUICKNODE_HTTP_URL="add-quicknode-http-provider-url-here"

PRIVATE_KEY="add-the-private-key-here"
Let's deploy the contract to the goerli network. Create a new file, or replace the default file, named deploy.js under the scripts folder

Let's write some code to deploy the contract in deploy.js file.

const { ethers } = require("hardhat");
exiger("dotenv").config({chemin:".env"});   
const{WHITELIST_CONTRACT_ADDRESS,METADATA_URL}=exiger("../constants");      

asyncfonctionprincipal(){   
  // Adresse du contrat de liste blanche que vous avez déployé dans le module précédent
  const whitelistContract =WHITELIST_CONTRACT_ADDRESS; 
  // URL d'où nous pouvons extraire les métadonnées d'un Crypto Dev NFT
  const metadataURL =METADATA_URL; 
  /*
  A ContractFactory in éthers.js est une abstraction utilisée pour déployer de nouveaux contrats intelligents,
  donc cryptoDevsContract est une usine pour les exemples de notre contrat CryptoDevs.
  */
  const cryptoDevsContract =attendre éthers.getContractFactory("CryptoDevs"); 

  // déployer le contrat
  const déployéCryptoDevsContract =attendre cryptoDevsContract.déployer( 
    metadataURL,
    whitelistContract
  );

  // Attendez qu'il termine le déploiement
  attendre déployéCryptoDevsContract.déployé();

  // imprimer l'adresse du contrat déployé
  console.log(
    "Crypto Devs Contract Address:",
    deployedCryptoDevsContract.address
  );
}

// Call the main function and catch if there is any error
main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
Comme vous pouvez le voir, deploy.js nécessite quelques constantes. Créons un dossier nommé constants sous le hardhat-tutorial dossier. Créer un index.js fichier à l'intérieur du constants dossier et ajoutez les lignes suivantes au fichier. Remplacez "adresse du contrat de blanchiment" par l'adresse du contrat de blanchiment que vous avez déployé dans le didacticiel précédent. Pour Metadata_URL, il suffit de copier l'exemple qui a été fourni. Nous remplacerions cela plus loin dans le tutoriel.

// Adresse du contrat Whitelist que vous avez déployé
constWHITELIST_CONTRACT_ADDRESS="adresse du contrat de blanchiste";   
// URL pour extraire des métadonnées pour un Crypto Dev NFT
constMETADATA_URL="https://nft-collection-sneh1999.vercel.app/api/";   

module.exportations={WHITELIST_CONTRACT_ADDRESS,METADATA_URL};     
Maintenant, ouvrez le hardhat.config.js fichier, nous allons configurer le goerli réseau ici afin que nous puissions déployer notre contrat sur le réseau Goerli. Remplacez toutes les lignes du hardhat.config.js déposer avec les lignes ci-dessous données

exiger("@nomicfoundation / hardhat-toolbox");
exiger("dotenv").config({chemin:".env"});   

constQUICKNODE_HTTP_URL= processus.env.QUICKNODE_HTTP_URL;  
constPRIVATE_KEY= processus.env.PRIVATE_KEY;  

module.exportations={  
  solidité:"0,8,4", 
  réseaux:{ 
    goerli:{ 
      url:QUICKNODE_HTTP_URL, 
      comptes:[PRIVATE_KEY], 
    },
  },
};
Compiler le contrat, ouvrir un terminal pointant vers hardhat-tutorial répertoire et exécutez cette commande

compile npx hardhat
To deploy, open up a terminal pointing at hardhat-tutorial directory and execute this command

npx hardhat run scripts/deploy.js --network goerli
Save the Crypto Devs Contract Address that was printed on your terminal in your notepad, you would need it futher down in the tutorial.

Website
To develop the website we would be using React and Next Js. React is a javascript framework which is used to make websites and Next Js is built on top of React. First, You would need to create a new next app. Your folder structure should look something like

- NFT-Collection
       - hardhat-tutorial
       - my-app
To create this my-app, in the terminal point to NFT-Collection folder and type

npx create-next-app@latest
and press enter for all the questions

Maintenant, pour exécuter l'application, exécutez ces commandes dans le terminal

cd my-app
npm run dev
Maintenant, allez à http://localhost:3000, votre application doit être en cours d'exécution 🤘

Installons maintenant la bibliothèque Web3Modal ( https://github.com/Web3Modal/web3modal). Web3Modal est une bibliothèque facile à utiliser pour aider les développeurs à ajouter la prise en charge de plusieurs fournisseurs dans leurs applications avec une configuration simple et personnalisable. Par défaut, Web3Modal Library prend en charge les fournisseurs injectés tels que ( Metamask, Dapper, Gnosis Safe, Frame, Web3 Browsers, etc ), vous pouvez également configurer facilement la bibliothèque pour prendre en charge Portis, Fortmatic, Squarelink, Torus, Authereum, D'CENT Wallet et Arkane. Ouvrez un terminal pointant versmy-app répertoire et exécutez cette commande

npminstaller web3modal 
Dans le même terminal, installez également ethers.js

npminstaller éthers 
In your public folder, download this folder and all the images in it (Download Link). Make sure that the name of the downloaded folder is cryptodevs

Now go to styles folder and replace all the contents of Home.modules.css file with the following code, this would add some styling to your dapp:

.main {
  min-height: 90vh;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  font-family: "Courier New", Courier, monospace;
}

.footer {
  display: flex;
  rembourrage:2rem0;  
  border-top:1px solide #eaeaea; 
  justifier le contenu: centre;
  aligner les éléments: centre;
}

.image{ 
  largeur:70%; 
  hauteur:50%; 
  marge gauche:20%; 
}

.title{ 
  taille de police:2rem; 
  marge:2rem0;  
}

.description{ 
  hauteur de ligne:1; 
  marge:2rem0;  
  taille de police:1.2rem; 
}

.bouton{ 
  rayon de frontière:4px; 
  couleur de fond:bleu; 
  frontière: aucun;
  couleur:#ffffff; 
  taille de police:15px; 
  rembourrage:20px; 
  largeur:200px; 
  curseur: pointeur;
  marge inférieure:2%; 
}
@media(largeur max:1000px){   
  .main{ 
    largeur:100%; 
    flex-direction: colonne;
    justifier le contenu: centre;
    aligner les éléments: centre;
  }
}
Ouvrez votre fichier index.js sous le dossier des pages et collez le code suivant, une explication du code peut être trouvée dans les commentaires.

importation{Contrat, fournisseurs, utils }de"éthers";    
importationTêtede"prochain / tête";   
importationRéagir,{ useEffect, useRef, useState }de"réagir";    
importationWeb3Modalde"web3modal";   
importation{ abi,NFT_CONTRACT_ADDRESS}de"../constants";     
importationstylesde"../styles/Home.module.css";   

exportpar défautfonctionAccueil(){    
  // portefeuilleConnected garder une trace de la connexion ou non du portefeuille de l'utilisateur
  const[portefeuilleConnecté, setWalletConnected]=useState(faux);   
  // presaleStarted garde une trace de la mise en service ou non de la prévente
  const[presaleDémarré, setPresaleStarted]=useState(faux);   
  // presaleEnded garde une trace de la fin de la prévente
  const[presaleEnded, setPresaleEnded]=useState(faux);   
  // le chargement est réglé sur true lorsque nous attendons qu'une transaction soit extraite
  const[chargement, setLoading]=useState(faux);   
  // vérifie si le portefeuille MetaMask actuellement connecté est le propriétaire du contrat
  const[isOwner, setIsOwner]=useState(faux);   
  // tokenIdsMinted garde une trace du nombre de jetons qui ont été frappés
  const[tokenIdsMinted, setTokenIdsMinted]=useState("0");   
  // Créer une référence au Web3 Modal ( utilisé pour la connexion à Metamask ) qui persiste tant que la page est ouverte
  const web3ModalRef =useRef(); 

  /**
   * presaleMint: Mint an NFT pendant la prévente
   */
  constpresaleMint=async()= >{      
    essayer{ 
      // Nous avons besoin d'un signataire ici car il s'agit d'une transaction «écrite».
      const signataire =attendregetProviderOrSigner(vrai);  
      // Créer une nouvelle instance du contrat avec un signataire, qui permet
      // méthodes de mise à jour
      const nftContract =nouveauContrat(NFT_CONTRACT_ADDRESS, abi, signataire);  
      // appeler le presaleMint du contrat, seules les adresses blanchies pourraient frapper
      const tx =attendre nftContract.presaleMint({ 
        // valeur signifie le coût d'un crypto dev qui est "0,01" eth.
        // Nous analysons la chaîne `0.01` à l'éther à l'aide de la bibliothèque utils d'ethers.js
        valeur: utils.parseEther("0,01"),
      });
      setLoading(vrai);
      // attendez que la transaction soit extraite
      attendre tx.attendre();
      setLoading(faux);
      fenêtre.alerte("Vous avez réussi à frapper un Crypto Dev!");
    }attraper(err){   
      console.erreur(err);
    }
  };

  /**
   * publicMint: Mint an NFT après la prévente
   */
  constpublicMint=async()= >{      
    essayer{ 
      // Nous avons besoin d'un signataire ici car il s'agit d'une transaction «écrite».
      const signataire =attendregetProviderOrSigner(vrai);  
      // Créer une nouvelle instance du contrat avec un signataire, qui permet
      // méthodes de mise à jour
      const nftContract =nouveauContrat(NFT_CONTRACT_ADDRESS, abi, signataire);  
      // appeler la menthe du contrat pour frapper le Crypto Dev
      const tx =attendre nftContract.menthe({ 
        // valeur signifie le coût d'un crypto dev qui est "0,01" eth.
        // Nous analysons la chaîne `0.01` à l'éther à l'aide de la bibliothèque utils d'ethers.js
        valeur: utils.parseEther("0,01"),
      });
      setLoading(vrai);
      // attendez que la transaction soit extraite
      attendre tx.attendre();
      setLoading(faux);
      fenêtre.alerte("Vous avez réussi à frapper un Crypto Dev!");
    }attraper(err){   
      console.erreur(err);
    }
  };

  /*
      connectWallet: connecte le portefeuille MetaMask
    */
  constconnectWallet=async()= >{      
    essayer{ 
      // Obtenez le fournisseur de web3Modal, qui dans notre cas est MetaMask
      // Lorsqu'il est utilisé pour la première fois, il invite l'utilisateur à connecter son portefeuille
      attendregetProviderOrSigner(); 
      setWalletConnected(vrai);
    }attraper(err){   
      console.erreur(err);
    }
  };

  /**
   * startPresale: commence la prévente de la collection NFT
   */
  conststartPresale=async()= >{      
    essayer{ 
      // Nous avons besoin d'un signataire ici car il s'agit d'une transaction «écrite».
      const signataire =attendregetProviderOrSigner(vrai);  
      // Créer une nouvelle instance du contrat avec un signataire, qui permet
      // méthodes de mise à jour
      const nftContract =nouveauContrat(NFT_CONTRACT_ADDRESS, abi, signataire);  
      // appelez le startPresale à partir du contrat
      const tx =attendre nftContract.startPresale(); 
      setLoading(vrai);
      // attendez que la transaction soit extraite
      attendre tx.attendre();
      setLoading(faux);
      // définir la prévente a commencé à vrai
      attendrecheckIfPresaleStarted(); 
    }attraper(err){   
      console.erreur(err);
    }
  };

  /**
   * checkIfPresaleStarted: vérifie si la prévente a commencé par interroger le `presaleStarted`
   * variable dans le contrat
   */
  constcheckIfPresaleStarted=async()= >{      
    essayer{ 
      // Obtenez le fournisseur de web3Modal, qui dans notre cas est MetaMask
      // Pas besoin du signataire ici, car nous ne lisons que l'état de la blockchain
      const fournisseur =attendregetProviderOrSigner();  
      // Nous nous connectons au contrat à l'aide d'un fournisseur, nous ne ferons donc que
      // avoir un accès en lecture seule au contrat
      const nftContract =nouveauContrat(NFT_CONTRACT_ADDRESS, abi, fournisseur);  
      // appeler le presaleDémarré du contrat
      const _presaleDémarré =attendre nftContract.presaleDémarré(); 
      si(!_presaleDémarré){  
        attendregetOwner(); 
      }
      setPresaleStarted(_presaleDémarré);
      retour _presaleDémarré;
    }attraper(err){   
      console.erreur(err);
      retourfaux; 
    }
  };

  /**
   * checkIfPresaleEnded: vérifie si la prévente a pris fin en interrogeant le `presaleEnded`
   * variable dans le contrat
   */
  constcheckIfPresaleEnded=async()= >{      
    essayer{ 
      // Obtenez le fournisseur de web3Modal, qui dans notre cas est MetaMask
      // Pas besoin du signataire ici, car nous ne lisons que l'état de la blockchain
      const fournisseur =attendregetProviderOrSigner();  
      // Nous nous connectons au contrat à l'aide d'un fournisseur, nous ne ferons donc que
      // avoir un accès en lecture seule au contrat
      const nftContract =nouveauContrat(NFT_CONTRACT_ADDRESS, abi, fournisseur);  
      // appeler le prévente Terminé du contrat
      const _presaleEnded =attendre nftContract.presaleEnded(); 
      // _presaleEnded est un grand nombre, nous utilisons donc le lt ( moins que la fonction ) au lieu de `< ``
      // Date.now ( ) / 1000 renvoie l'heure actuelle en secondes
      // Nous comparons si l'horodatage _presaleEnded est inférieur à l'heure actuelle
      // ce qui signifie que la prévente est terminée
      const a fini = _presaleEnded.lt(Math.plancher(Date.maintenant()/1000));  
      si(a fini){  
        setPresaleEnded(vrai);
      }autre{  
        setPresaleEnded(faux);
      }
      retour a fini;
    }attraper(err){   
      console.erreur(err);
      retourfaux; 
    }
  };

  /**
   * getOwner: appelle le contrat pour récupérer le propriétaire
   */
  constgetOwner=async()= >{      
    essayer{ 
      // Obtenez le fournisseur de web3Modal, qui dans notre cas est MetaMask
      // Pas besoin du signataire ici, car nous ne lisons que l'état de la blockchain
      const fournisseur =attendregetProviderOrSigner();  
      // Nous nous connectons au contrat à l'aide d'un fournisseur, nous ne ferons donc que
      // avoir un accès en lecture seule au contrat
      const nftContract =nouveauContrat(NFT_CONTRACT_ADDRESS, abi, fournisseur);  
      // appeler la fonction propriétaire à partir du contrat
      const _propriétaire =attendre nftContract.propriétaire(); 
      // Nous allons maintenant demander au signataire d'extraire l'adresse du compte MetaMask actuellement connecté
      const signataire =attendregetProviderOrSigner(vrai);  
      // Obtenir l'adresse associée au signataire connecté à MetaMask
      const adresse =attendre signataire.getAddress(); 
      si(adresse.toLowerCase()= = = _propriétaire.toLowerCase()){   
        setIsOwner(vrai);
      }
    }attraper(err){   
      console.erreur(err.message);
    }
  };

  /**
   * getTokenIdsMinted: obtient le nombre de jetons qui ont été frappés
   */
  constgetTokenIdsMinted=async()= >{      
    essayer{ 
      // Obtenez le fournisseur de web3Modal, qui dans notre cas est MetaMask
      // Pas besoin du signataire ici, car nous ne lisons que l'état de la blockchain
      const fournisseur =attendregetProviderOrSigner();  
      // Nous nous connectons au contrat à l'aide d'un fournisseur, nous ne ferons donc que
      // avoir un accès en lecture seule au contrat
      const nftContract =nouveauContrat(NFT_CONTRACT_ADDRESS, abi, fournisseur);  
      // appeler les jetons du contrat
      const _tokenIds =attendre nftContract.tokenIds(); 
      //_tokenIds est un `grand nombre. Nous devons convertir le grand nombre en chaîne
      setTokenIdsMinted(_tokenIds.toString());
    }attraper(err){   
      console.erreur(err);
    }
  };

  /**
   * Renvoie un objet fournisseur ou signataire représentant le RPC Ethereum avec ou sans
   * capacités de signature du métamas joint
   *
   * Un `fournisseur` est nécessaire pour interagir avec la blockchain - transactions de lecture, soldes de lecture, état de lecture, etc.
   *
   * Un `signeur` est un type spécial de fournisseur utilisé dans le cas où une transaction `write` doit être effectuée sur la blockchain, qui implique le compte connecté
   * besoin de faire une signature numérique pour autoriser l'envoi de la transaction. Metamask expose une API Signer pour permettre à votre site Web de
   * demander des signatures à l'utilisateur à l'aide des fonctions Signer.
   *
   * @param { * } needSigner - True si vous avez besoin du signataire, par défaut faux sinon
   */
  constgetProviderOrSigner=async(needSigner =faux)= >{       
    // Se connecter à Metamask
    // Puisque nous stockons `web3Modal` comme référence, nous devons accéder à la valeur `current` pour accéder à l'objet sous-jacent
    const fournisseur =attendre web3ModalRef.actuel.connecter(); 
    const web3Provider =nouveaufournisseurs.Web3Provider(fournisseur);  

    // Si l'utilisateur n'est pas connecté au réseau Goerli, faites-le savoir et lancez une erreur
    const{ chainId }=attendre web3Provider.getNetwork();   
    si(chainId != =5){   
      fenêtre.alerte("Changer le réseau en Goerli");
      lancernouveauErreur("Changer de réseau pour Goerli");  
    }

    si(needSigner){  
      const signataire = web3Provider.getSigner();
      retour signataire;
    }
    retour web3Provider;
  };

  // useEffects sont utilisés pour réagir aux changements d'état du site Web
  // Le tableau à la fin de l'appel de fonction représente les changements d'état qui déclencheront cet effet
  // Dans ce cas, chaque fois que la valeur de `walletConnected` change - cet effet sera appelé
  useEffect(()= >{  
    // si le portefeuille n'est pas connecté, créez une nouvelle instance de Web3Modal et connectez le portefeuille MetaMask
    si(!portefeuilleConnecté){  
      // Attribuez la classe Web3Modal à l'objet de référence en définissant sa valeur `current`
      // La valeur `current` persiste tant que cette page est ouverte
      web3ModalRef.actuel=nouveauWeb3Modal({   
        réseau:"goerli", 
        fournisseurOptions:{}, 
        disableInjectedProvider:faux, 
      });
      connectWallet();

      // Vérifiez si la prévente a commencé et s'est terminée
      const _presaleDémarré =checkIfPresaleStarted(); 
      si(_presaleDémarré){  
        checkIfPresaleEnded();
      }

      getTokenIdsMinted();

      // Définissez un intervalle qui est appelé toutes les 5 secondes pour vérifier la prévente terminée
      const presaleEndedInterval =setInterval(asyncfonction(){    
        const _presaleDémarré =attendrecheckIfPresaleStarted();  
        si(_presaleDémarré){  
          const _presaleEnded =attendrecheckIfPresaleEnded();  
          si(_presaleEnded){  
            effacer l'intervalle(presaleEndedInterval);
          }
        }
      },5*1000);   

      // définir un intervalle pour obtenir le nombre d'identifiants symboliques frappés toutes les 5 secondes
      setInterval(asyncfonction(){   
        attendregetTokenIdsMinted(); 
      },5*1000);   
    }
  },[portefeuilleConnecté]); 

  /*
      rend Button: Renvoie un bouton basé sur l'état du dapp
    */
  constrenderButton=()= >{     
    // Si le portefeuille n'est pas connecté, renvoyez un bouton qui leur permet de connecter leur wllet
    si(!portefeuilleConnecté){  
      retour( 
        <bouton surCliquer={connectWallet} nom de classe={styles.bouton}>
          Se connecter votre portefeuille
        </bouton>
      );
    }

    // Si nous attendons actuellement quelque chose, retournez un bouton de chargement
    si(chargement){  
      retour<nom de bouton={styles.bouton}>Chargement...</bouton>; 
    }

    // Si l'utilisateur connecté est le propriétaire et que la prévente n'a pas encore commencé, permettez-leur de démarrer la prévente
    si(isOwner &&!presaleDémarré){   
      retour( 
        <nom de bouton={styles.bouton} surClick={startPresale}>
          DémarrerPrésidence! 
        </bouton>
      );
    }

    // Si l'utilisateur connecté n'est pas le propriétaire mais que la prévente n'a pas encore commencé, dites-lui que
    si(!presaleDémarré){  
      retour( 
        <div>
          <div className={styles.description}>Présidence n'a pas commencé!</div>
        </div>
      );
    }

    // Si la prévente a commencé, mais n'est pas encore terminée, autorisez la frappe pendant la période de prévente
    si(presaleDémarré &&!presaleEnded){   
      retour( 
        <div>
          <div className={styles.description}>
            Présidence a commencé!!!Si votre adresse est blanchie,Menthe un Crypto  
            Dev 🥳
          </div>
          <nom de bouton={styles.bouton} surClick={presaleMint}>
            PrésidenceMenthe 🚀 
          </bouton>
        </div>
      );
    }

    // Si la prévente a commencé et s'est terminée, il est temps de frapper le public
    si(presaleDémarré && presaleEnded){  
      retour( 
        <nom de bouton={styles.bouton} surClick={publicMint}>
          PublicMenthe 🚀 
        </bouton>
      );
    }
  };

  retour( 
    <div>
      <Tête>
        <titre>CryptoDevs</titre> 
        <méta nom="description" contenu="Whitelist-Dapp"/> 
        <lien rel="icône" href="/favicon.ico"/> 
      </Tête>
      <div className={styles.principal}>
        <div>
          <h1 className={styles.titre}>Bienvenue à CryptoDevs!</h1> 
          <div className={styles.description}>
            Son un NFT collection pour développeurs dansCrypto. 
          </div>
          <div className={styles.description}>
            {tokenIdsMinted}/20 ont été frappés
          </div>
          {renderButton()}
        </div>
        <div>
          <img className={styles.image} src="./cryptodevs/0.svg"/> 
        </div>
      </div>

      <footer className={styles.pied de page}>
        Fabriquéavec&#10084; par CryptoDevs   
      </pied de page>
    </div>
  );
}
Maintenant, créez un nouveau dossier sous le my-app dossier et nom constants. Dans le constants dossier créer un fichier, index.js et collez le code suivant.

Remplacer "address of your NFT contract" avec l'adresse du contrat CryptoDevs que vous avez déployé et enregistré sur votre bloc-notes. Remplacer ---your abi--- avec l'abi de votre contrat CryptoDevs. Pour obtenir l'abi pour votre contrat, allez à votre hardhat-tutorial/artifacts/contracts/CryptoDevs.sol dossier et depuis votre CryptoDevs.json fichier obtenir le tableau marqué sous le "abi" clé.

exportconst abi =---ton abi--- 
exportconstNFT_CONTRACT_ADDRESS="adresse de votre contrat NFT"    
Maintenant dans votre terminal qui pointe vers my-app dossier, exécuter

npm run dev
Votre dapp NFT Crypto Devs devrait maintenant fonctionner sans erreurs 🚀

Pousser au github
Assurez-vous qu'avant de poursuivre, vous avez poussé tout votre code à github :)

Déploiement de votre dApp
Nous allons maintenant déployer votre dApp, afin que tout le monde puisse voir votre site Web et que vous puissiez le partager avec tous vos amis LearnWeb3 DAO.

Accédez à https://vercel.com/ et connectez-vous avec votre GitHub
Cliquez ensuite sur New Project bouton puis sélectionnez votre repo NFT-Collection
Lors de la configuration de votre nouveau projet, Vercel vous permettra de personnaliser votre Root Directory
Cliquez sur Edit à côté de Root Directory et le mettre à my-app
Sélectionnez le cadre comme Next.js
Cliquez sur DeployImage
Vous pouvez maintenant voir votre site Web déployé en allant dans votre tableau de bord, en sélectionnant votre projet et en copiant le domain de là! Enregistrer le domain sur le bloc-notes, vous en auriez besoin plus tard.

Voir votre collection sur Opensea
Maintenant, assurons-nous que votre collection est disponible sur Opensea

Pour rendre la collection disponible sur Opensea, nous devons créer un point final de métadonnées. Ce critère d'évaluation renverrait les métadonnées d'un NFT compte tenu de son tokenId.

Ouvre ton my-app dossier et souspages/api dossier, créez un nouveau fichier nommé [tokenId].js( Assurez-vous que le nom a également les parenthèses ). L'ajout des parenthèses permet de créer des itinéraires dynamiques dans js suivants

Ajouter les lignes suivantes à [tokenId].js fichier. Lisez à propos de l'ajout de routes API dans next js ici

exportpar défautfonctiongestionnaire(req, res){    
  // obtenir le jeton de la requête params
  const tokenId = req.requête.tokenId;
  // Comme toutes les images sont téléchargées sur github, nous pouvons extraire les images de github directement.
  const image_url =
    "https://raw.githubusercontent.com/LearnWeb3DAO/NFT-Collection/main/my-app/public/cryptodevs/";
  // L'api renvoie des métadonnées pour un Crypto Dev
  // Pour rendre notre collection compatible avec Opensea, nous devons suivre certaines normes de métadonnées
  // lors du renvoi de la réponse de l'api
  // Plus d'informations peuvent être trouvées ici: https://docs.opensea.io/docs/metadata-standards
  res.statut(200).json({
    nom:"Crypto Dev #"+ tokenId,  
    description:"Crypto Dev est une collection de développeurs en crypto", 
    image: image_url + tokenId +".svg", 
  });
}
Vous avez maintenant un itinéraire API qu'OpenSea et d'autres sites Web peuvent appeler pour récupérer les métadonnées du NFT. Ces sites Web appellent d'abord tokenURI sur le contrat intelligent pour obtenir le lien entre l'emplacement des métadonnées NFT. tokenURI leur donnera votre itinéraire API que nous venons de créer. Ensuite, ils peuvent appeler cette route API pour obtenir le nom, la description et l'image du NFT.

Déployons une nouvelle version de la Crypto Devs contrat avec cette nouvelle route API comme votre METADATA_URL

Ouvre ton hardhat-tutorial/constants dossier et à l'intérieur de votre index.js fichier, remplacez "https://nft-collection-sneh1999.vercel.app/api/" par le domaine que vous avez enregistré sur le bloc-notes et ajoutez" / api / "à sa fin.

Enregistrez le fichier et ouvrez un nouveau terminal pointant vers hardhat-tutorial dossier et déployer un nouveau contrat

npx hardhat run scripts / deploy.js --network goerli
Enregistrez la nouvelle adresse de contrat NFT sur un bloc-notes car nous en aurons besoin plus tard. Ouvrez le dossier "my-app / constantes" et à l'intérieur du index.js fichier remplacer l'ancienne adresse du contrat NFT par la nouvelle

Poussez tout le code sur github et attendez que Vercel déploie le nouveau code. Après que vercel a déployé votre code, ouvrez votre site Web et créez un NFT. Après la réussite de votre transaction, dans votre navigateur, ouvrez ce lien en remplaçant your-nft-contract-address avec l'adresse de votre contrat NFT ( https://testnets.opensea.io/assets/goerli/your-nft-contract-address/1)

Votre NFT est maintenant disponible sur Opensea 🚀 🥳 Partagez votre lien Opensea avec tout le monde sur la discorde: ) et répandez le bonheur.