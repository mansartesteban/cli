# Usage

Un utilitaire pour récupérer les valeurs des options de lignes de commande. Il permet également de créer un documentation rapide à attacher à chaque option.

Il sera alors possible de définir des options :

```javascript
const CLI = require("@/lib/CLI");

CLI.defineOption(["path", "p", "chemin", "c"], process.cwd(), "directory");
CLI.process();
```

Et en lançant la commande suivante par exemple :

```
node mon-script.js --path="chemin/de/destination"
```

Il sera possible d'accéder à la valeur du paramètre comme ceci :

```javascript
console.log(CLI.options.path); // "chemin/de/destination"
```

Il est aussi possible d'ajouter une documentation qui sera accessible via l'option `--help` ou `-h` et affichera automatiquement l'aide

```
Usage: mon-script.js <options>

Options :

--path, -p         Specifie le chemin d'accès à traiter
```

# Utilisation

## Import

Importer le fichier de cette manière :

```javascript
const CLI = require("@/lib/CLI");
```

## Options

Pour définir une option, il faudra utiliser la fonction `defineOption` :

```javascript
CLI.defineOption(["p", "path"], process.cwd, "directory");
```

Une fois tous les options définies il faudra indiquer à CLI de commencer le traitement via la fonction `process`

```javascript
// Si aucun paramètre n'est passé, utilisera process.argv (les paramètres de la commande executée via le terminal)
CLI.process();
CLI.process(["-ma", "liste", "--dargument"]);
```

Enfin, une fois ceci fait, on pourra récupérer la valeur des options via la propriété `options` pour les récupérer toutes ou la methode `getOption` pour les récupérer via leur nom

```javascript
console.log(CLI.options);

// OU

console.log(CLI.getOption("path"));
```

## Exemple complet

```javascript
const CLI = require("@/lib/CLI");

CLI.defineOption(["path", "p", "chemin", "c"], process.cwd(), "directory");
CLI.defineOption(["mode", "m"], "prod", "mode");
CLI.defineOption(["verbose"], false, "verbose");
CLI.defineOption("v", false, "version");
CLI.defineOption("f", false, "fetch");
CLI.process();

console.table(CLI.options);
console.log("Chemin du dossier", CLI.getOption("directory"));
```

Voici les résultats du code précédent :

`node maCommand --path=chemin/de/destination`

| (index)   | Values                  |
| --------- | ----------------------- |
| directory | 'chemin/de/destination' |
| verbose   | false                   |
| version   | false                   |
| fetch     | false                   |

`node maCommand -p chemin/de/destination/court`

| (index)   | Values                        |
| --------- | ----------------------------- |
| directory | 'chemin/de/destination/court' |
| verbose   | false                         |
| version   | false                         |
| fetch     | false                         |

`node maCommand -p chemin/vers/le/dossier/1 --path=chemin/vers/le/dossier/2`

| (index)   | Values                     |
| --------- | -------------------------- |
| directory | 'chemin/vers/le/dossier/1' |
| verbose   | false                      |
| version   | false                      |
| fetch     | false                      |

`node maCommand --verbose`

| (index) | Values |
| ------- | ------ |
| verbose | true   |
| version | false  |
| fetch   | false  |

`node maCommand -v`

| (index) | Values |
| ------- | ------ |
| verbose | false  |
| version | true   |
| fetch   | false  |

`node maCommand -mode development -chemin mon/chemin/vers/admin -vf --verbose --autreOption --p=autre/chemin/de/destination`

| (index)   | Values                  |
| --------- | ----------------------- |
| directory | 'mon/chemin/vers/admin' |
| mode      | 'development'           |
| verbose   | true                    |
| version   | true                    |
| fetch     | true                    |

## Ajouter une description

Pour ajouter une description il faudra utiliser la fonction `attachDescription` de cette manière :

```javascript
CLI.attachDescription("path", {
  en: "The path of the directory to be processed",
  fr: "Le chemin du dossier à traiter",
});
```

# API

## `CLI.launch(argv = []);`

Lance l'exécution du script avec les options paramétrées. Si `argv` n'est pas renseigné, utilisera `process.argv`

| nom  | type  | required | default | description           |
| ---- | ----- | -------- | ------- | --------------------- |
| argv | array | false    | []      | Les options à traiter |

## `CLI.reset();`

Permet de réinitialiser toutes les données de la classe.

## `CLI.getOption(name = "");`

Permet de récupérer la valeur d'une option par son nom

| nom  | type   | required | default | description                    |
| ---- | ------ | -------- | ------- | ------------------------------ |
| name | string | true     | ""      | Le nom de l'option à retourner |

## `CLI.defineOption([], "", "");`

Permet de définir une option qui sera calculée selon les paramètres envoyés dans la commande

| nom           | type     | required | default | description                                                                                                                                                                                                                                                                        |
| ------------- | -------- | -------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| tokens        | string[] | true     | []      | Les identifiants à interpéter sous une même option (ex: pour la valeur `["p", "path"]` alors l'option -p ou --path sera considéré comme le même option). N.B. Si plusieurs tokens pour la même option sont passées en ligne de commande, alors la valeur sera celle de la première |
| default_value | any      | true     | ""      | La valeur par défaut si l'option n'existe pas ou que sa valeur n'est pas interprétable                                                                                                                                                                                             |
| name          | string   | true     | ""      | Le nom de la propriété dans laquelle sera stockée la valeur de cet option                                                                                                                                                                                                          |

## `CLI.attachDescription(option_name = "", description = null);`

Permet d'attacher une description à l'option concernée

| nom         | type               | required | default | descrition                                                                                                                              |
| ----------- | ------------------ | -------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| option_name | string             | true     | ""      | The name of the option to attach the description                                                                                        |
| description | {[string]: string} | true     | null    | The description to give to the option. In key we have language codes (en, fr, pt), in values we have the corresponding description text |
