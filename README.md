# Développement d'une Application Web pour Suivre et Analyser des Investissements en Ethereum (ETH) et Bitcoin (BTC)

## Pertinence et Développement Personnel

### Compétences techniques
- **Technologies** : Développement en Angular (frontend) et Supabase (backend), deux technologies modernes et demandées.

### Domaine d'application
- **Secteur** : Le secteur des cryptomonnaies est en pleine expansion et offre de nombreuses opportunités.

### Apprentissage pratique
- **Objectif** : Construction d'une application concrète pour mieux assimiler les concepts théoriques.

### Portfolio
- **Avantage** : Excellent projet à ajouter à votre portfolio de développeur.

## Exemple Concret : Analyseur d'Investissement ETH/BTC

L'application permettra aux utilisateurs de :
- Visualiser les cours actuels et historiques de l'ETH et du BTC.
- Enregistrer leurs transactions (achats/ventes).
- Calculer le rendement de leur portefeuille.
- Visualiser des graphiques de performance.
- Configurer des alertes de prix personnalisées.

## Guide Étape par Étape

### Étape 1 : Mise en place de l'environnement de développement
1. **Installer Node.js et npm**
    - Télécharger depuis [nodejs.org](https://nodejs.org) (choisir la version LTS)
    - Vérifier l'installation :
    ```bash
    node --version
    npm --version
    ```
2. **Installer Angular CLI**
    ```bash
    npm install -g @angular/cli@next
    ng version
    ```
3. **Créer un nouveau projet Angular**
    ```bash
    ng new crypto-analyzer --strict
    ```
    - Répondre 'Yes' pour le routing Angular
    - Choisir 'SCSS' pour le format de feuille de style
4. **Naviguer dans le dossier du projet**
    ```bash
    cd crypto-analyzer
    ```
5. **Lancer le serveur de développement**
    ```bash
    ng serve
    ```
    - Ouvrir [http://localhost:4200](http://localhost:4200) dans le navigateur

### Étape 2 : Introduction à Supabase
1. **Créer un compte sur Supabase**
    - Aller sur [supabase.com](https://supabase.com)
    - Cliquer sur "Start your project"
    - S'inscrire avec email ou GitHub
2. **Créer un nouveau projet dans Supabase**
    - Cliquer sur "New project"
    - Nommer le projet (ex: "crypto-analyzer")
    - Choisir une région pour la base de données
    - Définir un mot de passe pour la base de données
    - Cliquer sur "Create new project"
    - Noter l'URL du projet et la clé API

### Étape 3 : Configuration de Supabase dans Angular
1. **Installer le SDK Supabase**
    ```bash
    npm install @supabase/supabase-js
    ```
2. **Créer un fichier `supabase.ts`**
    - Dans le dossier `src/app`, créer `supabase.ts`
3. **Configurer Supabase**
    ```typescript
    import { createClient } from '@supabase/supabase-js';
    
    const supabaseUrl = 'VOTRE_URL_SUPABASE';
    const supabaseKey = 'VOTRE_CLE_API_SUPABASE';

    export const supabase = createClient(supabaseUrl, supabaseKey);
    ```

### Étape 4 : Création de la structure de la base de données
1. **Dans l'interface Supabase, créer une table `transactions` :**
    - `id` (int8, primary key)
    - `user_id` (uuid, foreign key to auth.users)
    - `crypto` (text)
    - `amount` (numeric)
    - `price` (numeric)
    - `date` (timestamptz)
    - `type` (text) - 'buy' ou 'sell'

### Étape 5 : Authentification des utilisateurs
1. **Créer les composants d'authentification**
    ```bash
    ng generate component auth/signup
    ng generate component auth/login
    ```
2. **Implémenter l'inscription**
    - Dans `signup.component.ts` :
    ```typescript
    async signUp(email: string, password: string) {
      const { user, error } = await supabase.auth.signUp({ email, password });
      if (error) console.error('Erreur d\'inscription:', error.message);
      else console.log('Utilisateur inscrit:', user);
    }
    ```
3. **Implémenter la connexion**
    - Dans `login.component.ts` :
    ```typescript
    async signIn(email: string, password: string) {
      const { user, error } = await supabase.auth.signIn({ email, password });
      if (error) console.error('Erreur de connexion:', error.message);
      else console.log('Utilisateur connecté:', user);
    }
    ```

### Étape 6 : Création du Dashboard
1. **Créer le composant dashboard**
    ```bash
    ng generate component dashboard
    ```
2. **Afficher les cours actuels**
    - Utiliser l'API CoinGecko ([documentation](https://www.coingecko.com/en/api/documentation))
    - Créer un service pour les appels API :
    ```bash
    ng generate service services/crypto
    ```
    - Dans `crypto.service.ts` :
    ```typescript
    import { HttpClient } from '@angular/common/http';
    import { Injectable } from '@angular/core';

    @Injectable({
      providedIn: 'root'
    })
    export class CryptoService {
      constructor(private http: HttpClient) {}

      getPrices() {
        return this.http.get('https://api.coingecko.com/api/v3/simple/price?ids=bitcoin,ethereum&vs_currencies=usd');
      }
    }
    ```
3. **Créer un formulaire pour ajouter des transactions**
    - Utiliser Angular Reactive Forms
    - Dans `dashboard.component.ts` :
    ```typescript
    import { FormBuilder, FormGroup, Validators } from '@angular/forms';

    export class DashboardComponent {
      transactionForm: FormGroup;
      constructor(private fb: FormBuilder) {
        this.transactionForm = this.fb.group({
          crypto: ['', Validators.required],
          amount: ['', [Validators.required, Validators.min(0)]],
          price: ['', [Validators.required, Validators.min(0)]],
          type: ['', Validators.required]
        });
      }

      async addTransaction() {
        if (this.transactionForm.valid) {
          const { data, error } = await supabase
            .from('transactions')
            .insert([this.transactionForm.value]);
          if (error) console.error('Erreur:', error);
          else console.log('Transaction ajoutée:', data);
        }
      }
    }
    ```
4. **Afficher la liste des transactions**
    - Dans `dashboard.component.ts` :
    ```typescript
    async ngOnInit() {
      const { data, error } = await supabase
        .from('transactions')
        .select('*')
        .order('date', { ascending: false });
      if (error) console.error('Erreur:', error);
      else this.transactions = data;
    }
    ```

### Étape 7 : Analyse des investissements
1. **Créer un service d'analyse**
    ```bash
    ng generate service investment-analysis
    ```
2. **Implémenter les méthodes d'analyse**
    - Dans `investment-analysis.service.ts` :
    ```typescript
    @Injectable({
      providedIn: 'root'
    })
    export class InvestmentAnalysisService {
      calculateProfit(transactions: any[], currentPrices: any): number {
        // Logique de calcul du profit
      }

      calculateROI(transactions: any[], currentPrices: any): number {
        // Logique de calcul du ROI
      }

      calculatePerformancePerCrypto(transactions: any[], currentPrices: any): any {
        // Logique de calcul de la performance par crypto
      }
    }
    ```
3. **Afficher les analyses dans le dashboard**
    - Utiliser le service dans `dashboard.component.ts`
    - Afficher les résultats dans le template

### Étape 8 : Visualisation des données
1. **Installer `ng2-charts`**
    ```bash
    npm install ng2-charts chart.js
    ```
2. **Créer des graphiques**
    - Dans `dashboard.component.ts` :
    ```typescript
    import { ChartOptions, ChartType, ChartDataSets } from 'chart.js';
    import { Label } from 'ng2-charts';

    // ... Dans la classe DashboardComponent
    public lineChartData: ChartDataSets[] = [
      { data: [65, 59, 80, 81, 56, 55, 40], label: 'Portfolio Value' },
    ];
    public lineChartLabels: Label[] = ['January', 'February', 'March', 'April', 'May', 'June', 'July'];
    public lineChartOptions: ChartOptions = {
      responsive: true,
    };
    public lineChartType: ChartType = 'line';
    ```
    - Dans le template, ajouter :
    ```html
    <canvas baseChart
      [datasets]="lineChartData"
      [labels]="lineChartLabels"
      [options]="lineChartOptions"
      [chartType]="lineChartType">
    </canvas>
    ```

### Étape 9 : Système d'alertes
1. **Créer une table `alerts` dans Supabase**
    - `price_threshold` (numeric)
    - `crypto` (text)
    - `user_id` (uuid)
2. **Implémenter un formulaire pour créer des alertes**
    - Similaire au formulaire de transaction, mais pour les alertes
3. **Utiliser Supabase Realtime**
    ```typescript
    const subscription = supabase
      .from('alerts')
      .on('INSERT', payload => {
        console.log('Nouvelle alerte:', payload.new);
      })
      .subscribe();
    ```

### Étape 10 : Optimisation et Déploiement
1. **Optimiser les performances**
    - Utiliser le lazy loading des modules
    - Optimiser les requêtes Supabase
2. **Déployer l'application**
    - Choisir une plateforme (ex: Vercel, Netlify)
    - Configurer le déploiement continu depuis votre repository Git

## Conclusion
Ce projet vous permettra de développer une application complète en utilisant Angular et Supabase, tout en vous familiarisant avec le développement d'applications liées aux cryptomonnaies. Chaque étape vous fera progresser dans votre apprentissage du développement web moderne.
