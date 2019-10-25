- [Objectif](#org4f21b52)
- [Avec ou sans objets / classes](#orgc38a482)
  - [Astuce](#org853ba52)
  - [Tri des mots par nombre d'occurrences](#orgb4e7366)
- [Interactions](#org11d5bd9)
  - [Lister les textes](#org1fb4fb6)
  - [Ajouter un texte](#org36775c7)
  - [Supprimer un texte](#orgf7cb909)
  - [Action menant aux actions spécifiques à un texte](#org4903497)
  - [Sélectionner un texte dans une liste](#orgfab7e86)
  - [Afficher le nombre de lignes](#org5f18013)
  - [Afficher le nombre de mots](#orgdcc0636)
  - [Mots les plus fréquents](#orgfa25132)
  - [Mots spécifiques à un texte](#orge0c9a29)
  - [Pourcentage de mots partagés](#org8e9e379)
  - [Quitter](#orgcab53df)
  - [Menu](#org330a10d)
- [Classes Métier](#org2d22f06)
  - [Ensemble des textes](#org7fa3909)
  - [Texte](#orgca0301c)
- [Programme principal](#org63cc67b)



<a id="org4f21b52"></a>

# Objectif

Explorer plusieurs pistes de solutions pour comprendre les tenants et aboutissants


<a id="orgc38a482"></a>

# Avec ou sans objets / classes

À chaque fois qu'on a des données qui sont manipulées ensemble, on peut les regrouper au sein d'objets instances de classes. Ça ne veut pas dire qu'il **faut** le faire : les classes ajoutent du code qui peut participer à la complexité du programme. On va pouvoir décomposer / composer différentes classes pour implémenter le programme. Dans une approche descendante (*top-down*), on partira du niveau le plus global et l'on procède par décomposition. Dans une approche ascendante (*bottom-up*), on identifiera d'abord les "briques de base" du domaine et l'on composera celles-ci jusqu'à obtenir le niveau global. En pratique, on alternera souvent en faisant des aller-retour entre les différents niveaux <sup><a id="fnr.1" class="footref" href="#fn.1">1</a></sup>. Ici, on pourrait identifier trois types d'entités du domaine traité (*business logic*:

-   **`CountedWord`:** Un couple (mot, nombre d'occurrences) dans un texte donné.
-   **`Book`:** Les informations relatives à un texte (nom du fichier, mots qu'il contient,…)
-   **`BooksLibrary`:** Les informations concernant l'ensemble des textes manipulés. Cette classe contiendra aussi l'ensemble des opérations qu'il est possible de faire sur ces données, implémentant les fonctionnalités demandées, mais sans faire d'entrées/sorties, ce qui permettra une réutilisation maximum de ces fonctionnalités.

La classe la plus évidente est la classe `Book` car il y aura manifestement autant d'instances que de textes et l'on aura un certain nombre d'opérations qui manipulent les textes.

Pour la classe `BooksLibrary`, comme il n'y en a qu'une seule instance, on pourrait se contenter d'utiliser des variables (locales ou de classe) de la classe qui implémente le programme principal. Utiliser une instance est plus "propre" à deux points de vue :

-   en terme d'extensibilité : on pourra réutiliser les classes en cas d'évolution vers une architecture de type client-serveur qui permettra à plusieurs utilisateurs, chacun ayant sa propre instance de bibliothèque. Plus vraisemblablement, on pourra aussi implémenter plus efficacement des tests en ayant un programme de test qui pourra manipuler différentes bibliothèques.
-   en terme d'implémentation des interactions, on pourra vouloir passer un objet bibliothèque aux méthodes qui permettront d'utiliser la bibliothèque.

Pour la classe `CountedWord`, l'utilité n'est pas aussi évidente, car leur utilité n'est que très locale : au moment de calculer les `n` (=50) mots les plus fréquents d'un texte. À ce moment, on pourrait utiliser cette classe pour deux choses :

-   compter les mots
-   trier les mots comptés

Le problème est que :

-   pour compter les mots efficacement, on voudra pouvoir utiliser un [Set<CountedWord>](https://docs.oracle.com/javase/9/docs/api/java/util/Set.html) avec donc une redéfinition de la méthode [equals](https://docs.oracle.com/javase/9/docs/api/java/lang/Object.html#equals-java.lang.Object-) qui ne prenne pas en compte le nombre d'occurrences.
-   pour tirer les mots simplement, on voudra pouvoir utiliser [une méthode de tri](https://docs.oracle.com/javase/7/docs/api/java/util/Collections.html#sort(java.util.List)) qui prennent en compte le nombre nombre d'occurrences.


<a id="org853ba52"></a>

## Astuce

On pourrait être astucieux et réaliser que l'égalité et l'*ordre naturel* sont implémentés par deux méthodes distinctes [equals](https://docs.oracle.com/javase/9/docs/api/java/lang/Object.html#equals-java.lang.Object-) définie dans [Object](https://docs.oracle.com/javase/10/docs/api/java/lang/Object.html) et à redéfinir, et [compareTo](https://docs.oracle.com/javase/10/docs/api/java/lang/Comparable.html#compareTo(T)) déclarée dans [Comparable](https://docs.oracle.com/javase/10/docs/api/java/lang/Comparable.html) et qui doit donc être définie pour implémenter cette dernière.

Être astucieux en programmation peut cependant présenter des inconvénients et la documentation officielle indique :

> It is strongly recommended, but not strictly required that (x.compareTo(y)==0) == (x.equals(y)). Generally speaking, any class that implements the Comparable interface and violates this condition should clearly indicate this fact. The recommended language is "Note: this class has a natural ordering that is inconsistent with equals."

En effet, les astuces consistent souvent à tirer parti de conditions particulières et peuvent rendre le code 'fragile' si des évolutions remettent en cause ces dernières. Dans le cas du mini-projet en lui-même, il est facile de s'assurer qu'il n'y a pas de soucis avec cette astuce, mais on peut préférer s'habituer à éviter les astuces.


<a id="orgb4e7366"></a>

## Tri des mots par nombre d'occurrences

Si l'on renonce à l'astuce de l'implémentation de l'interface `Comparable` avec une méthode `compareTo` contredisant le `equals`, on perd beaucoup de l'intérêt d'une classe `CountedWord` qui ne serait alors plus utile deux fois (comptage et tri) mais une seule des deux fois, suivant que l'on prenne en compte les occurrences (pour le tri) ou non (pour le comptage).

Il est alors aussi simple d'utiliser des `String` et des `int` (et `Integer` pour stocker dans des structures de données) : si on doit de toutes façons utiliser un objet d'une classe implémentant [Comparator](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html) pour trier, on peut lui faire trier des mots de type `String` en prenant en compte les occurrences associées.

Si l'on voulait, on pourrait réutiliser les classes implémentant l'interface [Map.Entry<K,V>](https://docs.oracle.com/javase/8/docs/api/java/util/Map.Entry.html) (ici `Map.Entry<String,Integer>`, donc, contenues dans des `Map<String, Integer>`)pour manipuler des couples `(mot, nombre d'occurrence)` et on pourrait même les avoir dans l'ordre que l'on veut (i.e. triés selon le nombre d'occurrences) dans une [LinkedHashMap<K,V>](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html), mais une telle connaissance des structures de données en Java n'était pas attendue.


<a id="org11d5bd9"></a>

# Interactions

Pour le petit mini-projet, on peut se contenter de menus classiques en factorisant au moins le code d'affichage des choix et de saisie d'un entier valide. Si l'on veut une solution plus extensible dans une sorte de framework réutilisable, on peut définir des classes pour représenter les menus et les actions qu'ils permettent :

Une action effectuée à partir d'un menu sera caractérisée par un label (la ligne affichée dans le menu) et une action qui sera déclenchée lors de la sélection. Pour être le plus général possible, la méthode `doIt` qui implémente cette action ne prend pas d'arguments et ne retourne pas de résultat. Elle cependant accès aux éventuels attributs, ce qui lui permettra de récupérer des données qui auront été passées au constructeur de l'objet, et elle pourra avoir des effets de bords à travers des références (par exemple pour mettre des valeurs dans un tableau).

```java
public interface MenuItem {
    public String getLabel();
    public void doIt();
}
```


<a id="org1fb4fb6"></a>

## Lister les textes

```java
public class ListBooksAction implements MenuItem{
    private final static String LABEL= "Lister les fichiers";
    private final BooksLibrary library;

    public ListBooksAction (BooksLibrary library){
	this.library = library;
    }
    public String getLabel(){
	return LABEL;
    }
    public void doIt(){
	for(Book book : library.getBooks()){
	    System.out.println(book.getFilename());
	}
    }
}
```


<a id="org36775c7"></a>

## Ajouter un texte

```java
public class AddBookAction implements MenuItem{
    private final static String LABEL= "Ajouter un fichier";
    private final static String PROMPT= "Saisissez le nom du fichier à ajouter:";
    private final static String SUCCESS_MESSAGE= "Le fichier %s a bien été ajouté";
    private final static String FAILURE_MESSAGE= "Le fichier %s n'a pas pu être ajouté !";

    private final BooksLibrary library;

    public  AddBookAction (BooksLibrary library){
	this.library = library;
    }
    public String getLabel(){
	return LABEL;
    }
    public void doIt(){
	for(boolean done= false; !done; /* rien ici */) {
	    System.out.println(PROMPT);
	    String filename="";
	    try {
		filename = Main.sc.nextLine();
		library.add(new Book(filename));
		done= true; // Si on est ici, c'est que l'ajout a été fait
		System.out.println(String.format(SUCCESS_MESSAGE, filename));
	    }catch(Exception e){ // TODO préciser l'exception
		System.err.println(String.format(FAILURE_MESSAGE, filename));
		//e.printStackTrace();
	    }
	}
    }
}
```


<a id="orgf7cb909"></a>

## Supprimer un texte

```java
public class RemoveBookAction implements MenuItem{
    private final static String LABEL= "Supprimer un fichier";
    private final static String PROMPT= "Saisissez le nom du fichier à supprimer:";
    private final static String SUCCESS_MESSAGE= "Le fichier %s a bien été supprimé";
    private final static String FAILURE_MESSAGE= "Le fichier %s n'a pas pu être supprimé !";

    private final BooksLibrary library;

    public RemoveBookAction (BooksLibrary library){
	this.library = library;
    }
    public String getLabel(){
	return LABEL;
    }
    public void doIt(){
	if(library.getBooks().isEmpty())
	    { return ; } // on ne peut pas supprimer de texte car il n'y en a pas !
	for(boolean done= false; !done; /* rien ici */) {
	    System.out.println(PROMPT);
	    String filename= "";
	    try {
		filename = Main.sc.nextLine();
		library.remove(filename);
		done= true; // Si on est ici, c'est que l'ajout a été fait
		System.out.println(String.format(SUCCESS_MESSAGE, filename));
	    }catch(Exception e){
		System.err.println(String.format(FAILURE_MESSAGE, filename));
		//e.printStackTrace();
	    }
	}
    }
}
```


<a id="org4903497"></a>

## Action menant aux actions spécifiques à un texte

Cette action utilise elle-même deux menus :

1.  pour sélectionner le texte à traiter
2.  pour sélectionner l'action à effectuer sur ce texte

```java
import java.util.List;
import java.util.ArrayList;

public class BookSelectionAction implements MenuItem{

    private final static String LABEL = "Afficher les informations sur un livre";
    private final static String SELECT_BOOK_LABEL = "Veuillez choisir le texte de référence";
    private final static String SELECT_ACTION_LABEL = "Veuillez choisir l'action à effectuer";
    private final BooksLibrary library;

    public  BookSelectionAction (BooksLibrary library){
	this.library = library;
    }
    public String getLabel(){
	return LABEL;
    }
    private Book askSelectedBook(){
	List<MenuItem> entries = new ArrayList<MenuItem>();
	Book[] result = new Book[1];
	for(Book book : library.getBooks()){
	    entries.add(new SelectBookAction(book, result));
	}
	Menu menu= new Menu(SELECT_BOOK_LABEL, entries);
	menu.doIt();
	return result[0];
    }
    public void doIt(){
	Book book = askSelectedBook();
	List<MenuItem> entries = new ArrayList<MenuItem>();
	entries.add(new NbLinesAction(book));
	entries.add(new NbWordsAction(book));
	entries.add(new TopWordsAction(book));
	entries.add(new SpecificWordsAction(book, library));
	entries.add(new SharedWordsPercentAction(book, library));
	Menu menu= new Menu(SELECT_ACTION_LABEL, entries);
	menu.doIt();
    }
}
```


<a id="orgfab7e86"></a>

## Sélectionner un texte dans une liste

Cette action doit pouvoir "retourner" le livre sélectionné. Pour cela, on prend un tableau en argument du constructeur et on stocke la référence dans un attribut. Lorsque l'action est déclenchée, on modifie le contenu de ce tableau. Le code appelant, qui a passé la référence vers le tableau au constructeur et a lancé le menu, peut ensuite lire le contenu du tableau pour savoir quelle instance de `Book` a été sélectionnée.

```java
public class SelectBookAction implements MenuItem{
    private final Book book;
    private final Book[] result;

    public SelectBookAction (Book book, Book[] result){
	this.book = book;
	this.result = result; // TODO lancer une exception si result.length != 1 (ou au moins si < 1!)
    }
    public String getLabel(){
	return book.getFilename();
    }
    public void doIt(){
	result[0]= book;
    }
}
```


<a id="org5f18013"></a>

## Afficher le nombre de lignes

```java
public class NbLinesAction implements MenuItem{
    private final static String LABEL = "Afficher le nombre de lignes du fichier";
    private final static String MESSAGE = "Le fichier %s fait %d lignes";
    private final Book book;

    public NbLinesAction(Book book){
	this.book = book;
    }
    public String getLabel(){
	return LABEL;
    }
    public void doIt(){
	System.out.println(String.format(MESSAGE, book.getFilename(), book.getNbLines()));
    }
}
```


<a id="orgdcc0636"></a>

## Afficher le nombre de mots

Les spécifications n'étaient pas très explicite sur le nombre de mots (nombre de mots uniques) ou nombre de mots dans le fichier. Nous avons choisi d'implémenter le second cas (c'est de toutes façons délégué à la classe `Book`, mais ceci explique qu'on ne se contente pas de récupérer l'ensemble des mots et d'appeler `size` sur cet ensemble).

```java
public class NbWordsAction implements MenuItem{
    private final static String LABEL = "Afficher le nombre de mots du fichier";
    private final static String MESSAGE = "Le fichier %s contient %d mots";
    private final Book book;

    public NbWordsAction(Book book){
	this.book = book;
    }
    public String getLabel(){
	return LABEL;
    }
    public void doIt(){
	System.out.println(String.format(MESSAGE, book.getFilename(), book.getNbWords()));
    }
}
```


<a id="orgfa25132"></a>

## Mots les plus fréquents

```java
import java.util.List;

public class TopWordsAction implements MenuItem{
    public final static int NB_TOP_WORDS = 50;
    private final static String LABEL = "Afficher les mots les plus fréquents du fichier";
    private final static String MESSAGE = "Les mots les plus fréquents du fichier sont:";
    private final static String WORD_INFO = "%d : %s";
    private final Book book;

    public TopWordsAction(Book book){
	this.book = book;
    }
    public String getLabel(){
	return LABEL;
    }
    public void doIt(){
	System.out.println(MESSAGE);
	List<String> topWords = book.getTopWords();
	int end = Math.min(topWords.size(), NB_TOP_WORDS);
	for(int i=0; i != end; ++i){
	    String word= topWords.get(i);
	    System.out.println(String.format(WORD_INFO, book.getNbOcc(word), word));
	}
    }
}
```


<a id="orge0c9a29"></a>

## Mots spécifiques à un texte

```java
public class SpecificWordsAction implements MenuItem{
    private final static String LABEL = "Afficher les mots specifiques au fichier";
    private final static String MESSAGE = "Les mots spécifiques au fichier sont:";

    private final Book book;
    private final BooksLibrary library;

    public SpecificWordsAction(Book book, BooksLibrary library){
	this.book = book;
	this.library = library;
    }
    public String getLabel(){
	return LABEL;
    }
    public void doIt(){
	System.out.println(MESSAGE);
	for(String word : library.getSpecificWords(book)){
	    System.out.println(word);
	}
    }
}
```


<a id="org8e9e379"></a>

## Pourcentage de mots partagés

```java
import java.util.Map;

public class SharedWordsPercentAction implements MenuItem{
    private final static String LABEL = "Afficher les pourcentages de mots partagés avec ce fichier";
    private final static String MESSAGE = "Les pourcentages sont :";
    private final static String PERCENT_MESSAGE = "%s : %f%%";
    private final Book book;
    private final BooksLibrary library;

    public SharedWordsPercentAction(Book book, BooksLibrary library){
	this.book = book;
	this.library = library;
    }
    public String getLabel(){
	return LABEL;
    }
    public void doIt(){
	System.out.println(MESSAGE);
	for(Map.Entry<Book, Float> bookPercent : library.getSharedWordsPercent(book).entrySet()){
	    System.out.println(String.format(PERCENT_MESSAGE, bookPercent.getKey().getFilename()
						 , bookPercent.getValue()));
	}
    }
}
```


<a id="orgcab53df"></a>

## Quitter

On ne permet que le "O" majuscule lors de la demande de confirmation, on pourrait aussi accepter le 'o' minuscule.

```java
public class QuitAction implements MenuItem{
    private final static String LABEL = "Quitter le programme";
    private final static String CONFIRM_MESSAGE = "Êtes-vous sûr ? (%s / %s)";
    private final static String YES = "O";
    private final static String NO = "N";

    private final boolean[] result;

    public QuitAction(boolean[] result){
	this.result= result; // TODO lancer une exception si result.length != 1 (ou au moins si < 1 !)
    }
    public String getLabel(){
	return LABEL;
    }
    public void doIt(){
	System.out.println(String.format(CONFIRM_MESSAGE, YES, NO));
	result[0]= Main.sc.nextLine().equals(YES);
    }
}
```


<a id="org330a10d"></a>

## Menu

```java
import java.util.List;
import java.util.InputMismatchException;

public class Menu {
    private final static String PROMPT= "Saisissez le numéro de votre choix (entier entre 1 et %d):";
    private final static String ENTRY_MESSAGE = "%d. %s"; 
    private final static String ERROR_MESSAGE= "Saisie invalide";

    private String title;
    private final List<MenuItem> entries;
    
    public Menu(String title, List<MenuItem> entries){
	this.title = title;
	this.entries = entries;
    }
    /*retourn l'indice commençant à 0 et non la saisie (à partir de 1)
     */
    private int getChoice(){
	int res = -1;
	do{
	    try {
		System.out.println(String.format(PROMPT, entries.size()));
		res = Main.sc.nextInt();
		Main.sc.nextLine();
	    }catch(InputMismatchException e){
		System.err.println(ERROR_MESSAGE);
		Main.sc.nextLine();
	    }
	    }while((res < 1) || (res > entries.size()));
	return res - 1;
    }

    public void doIt(){
	System.out.println(title);
	for(int i= 0; i != entries.size(); ++i){
	    System.out.println(String.format(ENTRY_MESSAGE, i+1, entries.get(i).getLabel()));
	}
	entries.get(getChoice()).doIt();
    }
}
```


<a id="org2d22f06"></a>

# Classes Métier


<a id="org7fa3909"></a>

## Ensemble des textes

En terme de données, cette classe n'apporte rien par rapport à un ensemble d'instance de `Book`. Elle permet de regrouper les opérations sur cet ensemble de textes. Il est important de noter que ces opérations ne font pas d'entrées/sorties. Elles sont appelées par les méthodes qui font des E/S.

```java
import java.util.Set;
import java.util.HashSet;
import java.util.Map;
import java.util.HashMap;
import java.util.NoSuchElementException;

public class BooksLibrary{
    private final Set<Book> books = new HashSet<Book>();
    public BooksLibrary(){
    }
    public void add(Book book){
	books.add(book);
    }
    public void remove(Book book){
	books.remove(book);
    }
    public void remove(String filename){
	remove(getByFilename(filename));
    }
    public Book getByFilename(String filename){
	// TODO utiliser Optional http://www.touilleur-express.fr/2014/11/07/optional-en-java-8/
	for(Book book: books){
	    if(book.getFilename().equals(filename)){
		return book;
	    }
	}
	throw new NoSuchElementException(filename);
	//	return null; // !! cf Optional
    }
    public Set<Book> getBooks(){
	// TODO https://docs.oracle.com/javase/7/docs/api/java/util/Collections.html#unmodifiableSet(java.util.Set)
	return books;
    }
    public Set<String> getSpecificWords(Book book){
	Set<String> res = new HashSet<String>(book.getWords());
	for(Book otherBook : books){
	    if(otherBook != book){
		res.removeAll(otherBook.getWords());
	    }
	}
	return res;
    }
    public Map<Book, Float> getSharedWordsPercent(Book book){
	Map<Book, Float> res= new HashMap<Book, Float>();
	Set<String> words = book.getWords();
	for(Book otherBook : books){
	    if(otherBook != book){
		Set<String> otherWords = new HashSet<String>(otherBook.getWords());
		int nbWords = otherWords.size();
		otherWords.retainAll(words);
		res.put(otherBook, 100.f*otherWords.size()/nbWords);
	    }
	}
	return res;
    }
    public boolean equals(Object o){
	return (o != null) && (o instanceof BooksLibrary) && (books.equals(((BooksLibrary)o).books));
    }
    public int hashCode(){
	return books.hashCode();
    }
}
```


<a id="orgca0301c"></a>

## Texte

Si une fichier `.java` ne peut contenir qu'une seule classe *publique*, il peut contenir d'autres classes de visibilité réduite (i.e. la visibilité par défaut, réduite au *package*). On ajoute ici la classe qui permet de trier les mots en fonction du nombre d'occurrences. On utilise le fait que l'instance de cette classe peut avoir des attributs d'instance pour avoir une référence vers la table qui associe les nombre d'occurrences aux mots.

```java
import java.util.Collections;
import java.util.Map;
import java.util.HashMap;
import java.util.List;
import java.util.ArrayList;
import java.util.Set;
import java.util.Comparator;
import java.util.Scanner;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import java.io.FileNotFoundException;
import java.io.File;

public class Book{
    private final String filename;
    private Map<String, Integer> wordToNbOcc = new HashMap<String, Integer>();
    private long nbLines;
    private long nbWords;
    public Book(String filename) throws FileNotFoundException {
	this.filename= filename;
        Pattern p = Pattern.compile("\\w+", Pattern.UNICODE_CHARACTER_CLASS);
        try(Scanner sc = new Scanner(new File(filename))){
            for(nbLines = 0, nbWords = 0; sc.hasNextLine(); ++nbLines){
                for(Matcher m1 = p.matcher(sc.nextLine()); m1.find(); ++nbWords) {
		    String word= m1.group().toLowerCase();
		    Integer nbOcc= wordToNbOcc.get(word);
		    wordToNbOcc.put(word, (nbOcc == null) ? 1 : nbOcc+1);
                }
            }
        }
    }

    public String getFilename(){
	return filename;
    }
    public long getNbLines(){
	return nbLines;
    }
    public long getNbWords(){
	return nbWords;
    }
    public Set<String> getWords(){
	// https://docs.oracle.com/javase/9/core/creating-immutable-lists-sets-and-maps.htm
	// TODO https://docs.oracle.com/javase/7/docs/api/java/util/Collections.html#unmodifiableSet(java.util.Set)
	return wordToNbOcc.keySet();
    }
    public List<String> getTopWords(){
	List<String> res= new ArrayList<String>(wordToNbOcc.keySet());
	Collections.sort(res, new NbOccComparator(wordToNbOcc)); // devrait être une classe interne
	return res;
    }
    public int getNbOcc(String word){
	return wordToNbOcc.get(word);
    }
}

class NbOccComparator implements Comparator<String> {
    Map<String, Integer> wordToNbOcc;
    public NbOccComparator(Map<String, Integer> wordToNbOcc){
	this.wordToNbOcc = wordToNbOcc;
    }
    public int compare(String word1, String word2){
	int res= wordToNbOcc.get(word2).compareTo(wordToNbOcc.get(word1));
	return (res != 0) ? res : word1.compareTo(word2);
    }
}

```


<a id="org63cc67b"></a>

# Programme principal

Le programme principal est très réduit : il se contente d'initialiser le menu et d'appeler celui-ci en boucle tant que l'action `QuitAction` n'a pas modifié le tableau passé en argument de son constructeur pour indiquer la fin demandée.

En fait, il faudrait avoir un menu spécifique lorsque le nombre de textes est `0` car il n'est alors pas possible de supprimer un texte !

```java
import java.util.Scanner;
import java.util.List;
import java.util.ArrayList;
import java.util.InputMismatchException;

public class Main {
    private final static String TITLE = "Gestion de collection de textes :";
    private final static String LOADING_START = "Chargement de %s ...";
    private final static String LOADING_END = " effectué";
    public static final Scanner sc= new Scanner(System.in);

    public static void main(String[] args){
	BooksLibrary library = new BooksLibrary();
	for(String filename : args){
	    try{
		System.out.print(String.format(LOADING_START, filename));
		library.add(new Book(filename));
		System.out.println(LOADING_END);
	    }catch(Exception e){
		e.printStackTrace();
	    }
	}
	boolean [] quitting= new boolean[1];
	List<MenuItem> entries = new ArrayList<MenuItem>();
	
	entries.add(new ListBooksAction(library));
	entries.add(new AddBookAction(library));
	entries.add(new RemoveBookAction(library));
	entries.add(new BookSelectionAction(library));
	entries.add(new QuitAction(quitting));
	for(Menu menu= new Menu(TITLE, entries); !quitting[0]; menu.doIt()){
	}
    }
}
```

## Footnotes

<sup><a id="fn.1" class="footnum" href="#fnr.1">1</a></sup> Lorsqu'on débute, il est peut-être plus
facile de suivre une approche descendante car on part de ce qu'on
connaît le mieux : les spécifications qui concernent surtout le niveau
global. Avec l'expérience, on privilégiera souvent une approche
ascendante, l'intuition permettant d'anticiper les composants
élémentaires qui seront utiles.
