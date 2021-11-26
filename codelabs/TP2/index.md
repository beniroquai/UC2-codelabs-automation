---
id: TP2
source: tp/TP2.md
duration: 0

---

# TP 2 - Actions & Intents




## Suppression d'une tache



Dans le layout de votre ViewHolder, ajouter un `ImageButton` qui servira à supprimer la tâche associée. Vous pouvez utiliser par exemple l'icone `@android:drawable/ic_menu_delete`

**Rappel:** Une  [lambda](https://kotlinlang.org/docs/reference/lambdas.html) est un type de variable qui contient un bloc de code pouvant prendre des arguments et retourner un résultat, c'est donc une fonction que l'on utilise comme une variable !

Aidez vous des lignes de code plus bas pour réaliser un "Click Listener" à l'aide d'une lambda en suivant ces étapes:

* Dans l'adapter, ajouter une propriété lambda `onClickDelete` qui prends en arguments une `Task` et ne renvoie rien: `(Task) -&gt; Unit` et l'initier à `null` (elle ne fait rien par défaut)
* Utilisez cette lambda dans le `onClickListener` du bouton supprimer
* Dans le fragment, accéder à `onClickDelete` depuis l'adapter et implémentez là: donnez lui comme valeur une lambda qui va supprimer la tache passée en argument de la liste

```language-kotlin
// Déclaration de la variable lambda dans l'adapter:
var onClickDelete: ((Task) -> Unit)? = null

// "implémentation" de la lambda dans le fragment:
adapter.onClickDelete = { task ->
    // Supprimer la tâche
}

// Utilisation de la lambda dans le ViewHolder:
onClickDelete?.invoke(task)
```


## Ajout de tâche complet



* Créer un package `task`
* Créez y la nouvelle `TaskActivity`, n'oubliez pas de la déclarer dans le manifest
* Créer un layout contenant 2 `EditText`, pour le titre et la description et un bouton pour valider
* Changer l'action du FAB pour qu'il ouvre cette activité avec un `Intent`, en attendant un resultat:

```language-kotlin
val intent = Intent(activity, TaskActivity::class.java)
startActivityForResult(intent, ADD_TASK_REQUEST_ID)
```

* Dans le `onCreate` de la nouvelle activité, récupérer le bouton de validation puis setter son `onClickListener` pour qu'il crée une tâche:

```language-kotlin
// Instanciation d'un nouvel objet [Task]
val newTask = Task(id = UUID.randomUUID().toString(), title = "New Task !")
```

* Faites hériter `Task` de `Serializable` pour pouvoir passer des objets `Task` dans les `intent`
* Passez `newTask` dans l'intent avec `putExtra(...)`
* utlisez `setResult(...)` et `finish()` pour retourner à l'activité principale
* Dans celle ci, overrider `onActivityResult` dans le `TaskFragment` pour récupérer cette task et l'ajouter à la liste

```language-kotlin
val task = data?.getSerializableExtra(TaskActivity.TASK_KEY) as? Task
```

* Faites en sorte que la nouvelle tache s'affiche au retour sur l'activité principale
* Maintenant, récupérez les valeurs entrées dans les `EditText` pour les donner à la création de votre tâche (vous devrez faire un `toString()`)


## Édition d'une tâche



* Ajouter une bouton permettant d'éditer chaque tâche en ouvrant l'activité `TaskActivity` pré-remplie avec les informations de la tâche
* Pour transmettre des infos d'une activité à l'autre, vous pouvez utiliser la méthode `putExtra` depuis une instance d'`intent`
* Inspirez vous de l'implémentation du bouton supprimer et du bouton ajouter
* Vous pouvez ensuite récupérer dans le `onCreate` de l'activité les infos que vous avez passées:

* récupérez la tâche passée avec un `getSerializableExtra` et un `as? Task`
* Vous pourrez tirer parti de la variable `Task?` (nullable) pour réutiliser le code de la création et remplir les `EditText`
* De même vous pourrez utiliser l'opérateur `?:` pour setter l'`id` en utilisant la méthode `UUID...` précédente par défaut
* Utilisez `setText` pour préremplir les `EditText`
* Au retour dans `onActivityResult`, vous pouvez utiliser `indexOfFirst { /* une condition */ }` sur votre liste pour trouver la tache concernée et la remplacer dans la liste, et s'il n'y  a pas, c'est qu'il s'agit d'un ajout
* Vérifier que les infos éditées s'affichent bien à notre retour sur l'activité principale.


## Nouvelle API ActivityResult



Changez votre code pour utiliser cette nouvelle méthode, par ex avec `StartActivityForResult()`:

```language-kotlin
val startForResult = registerForActivityResult(StartActivityForResult()) {...}
// ...
startForResult.launch(intent)
```

Mais vous pouvez aussi définir un  [contrat spécifique](https://developer.android.com/training/basics/intents/result#custom):

```language-kotlin
class EditTask : ActivityResultContract<Task, Task>() {
    override fun createIntent(...)
    override fun parseResult(...)
}
```


## Partager



* En modifiant `AndroidManifest.xml`, ajouter la possibilité de partager du texte **depuis les autres applications** (par ex en surlignant un texte dans un navigateur puis en cliquant sur "partager") et ouvrir le formulaire de création de tâche avec une description pré-remplie ( [Documentation](https://developer.android.com/training/sharing/receive))
* En utilisant un `Intent` **implicite**, ajouter la possibilité de partager du texte **vers les autres applications** (avec un `OnLongClickListener` sur les tâches par ex ou bien avec un bouton dans la vue formulaire) ( [Documentation](https://developer.android.com/training/sharing/send))


## Changements de configuration



Que se passe-t-il si vous tournez votre téléphone pour passer l'app en mode paysage ? 🤔

* Une façon de régler ce soucis est d'overrider la méthode suivante:

```language-kotlin
override fun onSaveInstanceState(outState: Bundle)
```

* Il faudra utiliser `putParcelableArrayList`
* Il faudra aussi que votre classe `Task` hérite de `Parcelable`: pour implémenter  [automatiquement](https://developer.android.com/kotlin/parcelize) les méthodes nécessaires, ajoutez le plugin `kotlin-parcelize` à votre `app/build.gradle` et l'annotation `@Parcelize` à votre classe `Task`
* Puis, pour récupérer cette liste, utilisez l'argument `savedInstanceState` et la méthode `getParcelableArrayList` dans `onCreateView`


## Interface et délégation



Une façon plus classique de gérer les clicks d'un item est de définir une interface que l'on implémentera dans l'Activity/Fragment. Mettez à jour votre code pour utiliser cette méthode:

```language-kotlin
interface TaskListListener {
  fun onClickDelete(task: Task)
}

class TaskListAdapter(val listener: TaskListListener) : ... {
  // use: listener.onClickDelete(task)
}

class TaskListFragment : Fragment {
  val adapterListener = object : TaskListListener {
    override onClickDelete(task: Task) {...}
  }
  val adapter = TaskListAdapter(adapterListener)
}
```

Prenez exemple sur ceci pour remplacer vos lambdas.

