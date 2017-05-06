# Il était une fois une fonction `reduce`

Supposons une collection d'entiers:

    Collection<Integer> integers = Arrays.asList(1, 2, 3, 4)

Dans une approche impérative, lorsque nous voulons calculer une valeur à partir d'une liste de valeurs nous devons utiliser une boucle sur la liste de valeurs et accumuler le résultat à l'extérieur de la boucle. Par exemple pour calculer la somme des entiers nous procédons comme suit:

    int sum = 0;
    for (int i : integers) {
      sum += i;
    }

Dans une approche fonctionnelle nous déléguons ce traitement à une fonction nommée `reduce`. Pour sommer les entiers nous procédons comme ceci:

    reduce((sum, i) -> sum + i, 0, integers)

Le traitement à effectuer est passé en paramètre en définissant une <i> fonction de réduction </i> (`(sum, i) -&gt; sum + i`).

Savez-vous qu'avec cette <i> fonction de réduction </i> on peut définir les fonctions `map`et `filter`? On peut même combiner les `map`et `filter` pour former une <i> fonction de reduction </i> qui s'applique à une collection sans avoir de collection intermédiaire. Cette technique s'appelle les [`Transducers`](http://clojure.org/reference/transducers). Les [`Transducers`](http://clojure.org/reference/transducers) sont un concept qui vient de l'univers Clojure. Mais même en Clojure, tout le monde ne les connait pas. Je les ai découverts grâce aux excellentes présentations d'Arnaud Lemaire ([@lilobase](https://twitter.com/Lilobase)). Arnaud utilise des langages dynamiques non typés (python, PHP, etc...) pour sa démonstration. J'ai décider de relever le défi de coder les [`Transducers`](http://clojure.org/reference/transducers) en Java 8 avec ses types génériques qui ne sont pas toujours simples à dompter. Comme si cela ne suffit pas, je vais même montrer au passage comment ça marche, et même comment le concept est inévitable si comme un craftsman qui se respecte nous refactorisons régulièrement note code. Vous refactorisez régulièrement aussi n'est-ce pas?

Notre démarche consiste à l'aide d'un exemple métier concret (calculer le montant total des factures dues pour un mois donné) de partir d'une implémentation impérative, pour ensuite aller vers une implémentation fonctionnelle. On verra au passage comme écrire les fonctions `reduce`, `map` et `filter`. En refactorisant successivement notre fonction de calcul nous aboutirons au développement et à l'utilisation des [`Transducers`](http://clojure.org/reference/transducers).

## Une collection de facture

Pour faire cette démonstration nous utiliserons les factures suivantes:
<table><caption>Liste des factures</caption>
<tbody>
<tr>
<th>Due pour le mois</th>
<th>Montant H.T.</th>
<th>TVA</th>
<th>Quantité</th>
<th>Payée</th>
</tr>
<tr>
<td>Octobre</td>
<td>36.0</td>
<td>20 %</td>
<td>1</td>
<td>Non</td>
</tr>
<tr>
<td>Septembre</td>
<td>28.0</td>
<td>10 %</td>
<td>1</td>
<td>Oui</td>
</tr>
<tr>
<td>Octobre</td>
<td>17.0</td>
<td>5 %</td>
<td>2</td>
<td>Non</td>
</tr>
<tr>
<td>Octobre</td>
<td>27.0</td>
<td>5 %</td>
<td>2</td>
<td>Oui</td>
</tr>
</tbody>
</table>
En Java pour représenter le tableau ci-dessus nous construisons la collection de factures (`Invoice`) comme suit:

    static Collection<Invoice> invoices() {
        return Arrays.asList( //
                anInvoice(invoiceBuilder -> {
                    invoiceBuilder.dueTo(Month.OCTOBER);
                    invoiceBuilder.amountExcludeVAT(36.0);
                    invoiceBuilder.vatRate(20.0);
                    invoiceBuilder.quantity(1);
                    invoiceBuilder.mustBePaid();
                }), //
                anInvoice(invoiceBuilder -> {
                    invoiceBuilder.dueTo(Month.SEPTEMBER);
                    invoiceBuilder.amountExcludeVAT(28.0);
                    invoiceBuilder.vatRate(10.0);
                    invoiceBuilder.quantity(1);
                    invoiceBuilder.isPaid();
                }), //
                anInvoice(invoiceBuilder -> {
                    invoiceBuilder.dueTo(Month.OCTOBER);
                    invoiceBuilder.amountExcludeVAT(17.0);
                    invoiceBuilder.vatRate(5.0);
                    invoiceBuilder.quantity(2);
                    invoiceBuilder.mustBePaid();
                }), //
                anInvoice(invoiceBuilder -> {
                    invoiceBuilder.dueTo(Month.OCTOBER);
                    invoiceBuilder.amountExcludeVAT(27.0);
                    invoiceBuilder.vatRate(5.0);
                    invoiceBuilder.quantity(2);
                    invoiceBuilder.isPaid();
                }));
    }

Les factures sont définies comme suit:

    public class Invoice {
        private final Month dueTo;
        private final Amount amountExcludeVAT;
        private final Quantity quantity;
        private final Rate vatRate;
        private final boolean paid;

        public static Invoice anInvoice(Consumer<Builder> consumer) {
            Builder builder = new Builder();
            consumer.accept(builder);
            return builder.build();
        }

        private Invoice(Month dueTo, Amount amountExcludeVAT, Quantity quantity,
                        Rate vatRate, boolean paid) {
            this.dueTo = dueTo;
            this.amountExcludeVAT = amountExcludeVAT;
            this.quantity = quantity;
            this.vatRate = vatRate;
            this.paid = paid;
        }

        public boolean isDueFor(Month month) { return !paid && dueTo == month; }

        public Amount totalIncludingVAT() {
            return amountExcludeVAT.add(VATAmount()).multiply(quantity);
        }

        private Amount VATAmount() {
             return amountExcludeVAT.divideByHundred().multiply(vatRate);
        }

        public static class Builder {
            private Month dueMonth;
            private Amount amountExcludeVAT;
            private Quantity quantity;
            private Rate vatRate;
            private boolean paid;

            private Builder() {}

            public void dueTo(Month month) { dueMonth = month; }

            public void amountExcludeVAT(double amountExcludeVAT) {
                this.amountExcludeVAT = Amount.of(amountExcludeVAT);
            }

            public void quantity(long quantity) { this.quantity = Quantity.of(quantity); }

            public void vatRate(double vatRate) { this.vatRate = Rate.of(vatRate); }

            public void mustBePaid() { paid = false; }

            public void isPaid() { paid = true; }

            private Invoice build() {
                return new Invoice(dueMonth, amountExcludeVAT, quantity, vatRate, paid);
            }
        }
    }

    public class Amount {
        public static final Amount HUNDRED = Amount.of(100);
        public static final Amount ZERO = Amount.of(BigDecimal.ZERO);

        private final BigDecimal value;

        public static Amount of(double amountAsDouble) {
            return Amount.of(BigDecimal.valueOf(amountAsDouble));
        }

        public static Amount of(BigDecimal amountAsBigDecimal) {
            return new Amount(Objects.requireNonNull(amountAsBigDecimal));
        }

        private Amount(BigDecimal value) {
            this.value = value.setScale(2, BigDecimal.ROUND_DOWN);
        }

        public Amount add(Amount otherAmount) {
            return new Amount(value.add(otherAmount.value));
        }

        public Amount multiply(Quantity quantity) {
            return Amount.of(value.multiply(quantity.asBigDecimal()));
        }

        public Amount divideByHundred() {
            return Amount.of(value.divide(Amount.HUNDRED.value, 2, BigDecimal.ROUND_DOWN));
        }

        public Amount multiply(Rate vatRate) {
            return Amount.of(value.multiply(vatRate.asBigDecimal()));
        }
    }

    class Quantity  {
        private final long value;

        static Quantity of(long value) { return new Quantity(value); }

        private Quantity(long value) { this.value = value; }

        BigDecimal asBigDecimal() { return BigDecimal.valueOf(value); }

    }

    class Rate {
        private final double value;

        static Rate of(double value) { return new Rate(value); }

        private Rate(double value) { this.value = value; }

        BigDecimal asBigDecimal() { return BigDecimal.valueOf(value); }

    }

Nous allons calculer le montant total des factures dues pour le mois d'octobre. On commence donc par un test:

    assertThat(getAmountOf(Month.OCTOBER)).isEqualTo(Amount.of(78.90));

## Première implémentation en style impératif

    Amount getAmountOf(Month month) {
        Amount totalAmount = Amount.ZERO;

        for (Invoice invoice: invoices()) {
            if (invoice.isDueFor(month)) {
                totalAmount = totalAmount.add(invoice.totalIncludingVAT());
            }
        }
        return totalAmount;
    }

Cette implémentation se fait avec les étapes suivantes:
<ul>
 	<li>Nous définissons une variable `totalAmount` et nous l'initialisons avec la valeur initiale zéro.</li>
 	<li>Puis nous bouclons sur la liste des factures.</li>
 	<li>Pour chaque facture nous vérifions si elle est due pour le mois donné.</li>
 	<li>Si la facture est due:
<ul>
 	<li>Nous calculons son montant toutes taxes comprises;</li>
 	<li>nous ajoutons son montant toutes taxes comprises au total des factures dues.</li>
</ul>
</li>
</ul>
## Deuxième implémentation avec un `filter`, un `map` et un `reduce`

    Amount getAmountOf(Month month) {
        List<Invoice> invoicesOfTheMonth =
            filter(invoice -> invoice.isDueFor(month), invoices());
        List<Amount> includeVATAmounts =
            map(Invoice::totalIncludingVAT, invoicesOfTheMonth);
        return reduce(Amount::add, Amount.ZERO, includeVATAmounts);
    }

Avec cette implémentation nous avons les étapes suivantes:

* Sélectionner toutes les factures dues dans le mois spécifié (`filter`).
* Transformer la liste des factures dues en une liste de montants toutes taxes comprises (`map`).
* Calculer la somme des montants dus pour la liste de montants toutes taxes comprises (`reduce`).

Dans cette implémentation nous voyons que:

* Chaque étape à réaliser correspond à une déclaration (on définit ce que c'est et non comment on le fait).
* Les paramètres en entrée ne sont pas modifiés.
* Les étapes et comportements sont isolés.
* Les effets de bords sont supprimés.
* Les comportements peuvent être réutilisés.
* Le code est plus modulaire.

Cette implémentation pourrait être réaliser avec une version de Java inférieure à 8, en utilisant les classes anonymes :

    Amount getAmountOf(Month month) {
        List<Invoice> invoicesOfTheMonth = filter(new Predicate<Invoice>() {
            @Override
            public boolean test(Invoice invoice) {
                return invoice.isDueFor(month);
            }
        }, invoices());
        List<Amount> includeVATAmounts = map(new Function<Invoice, Amount>() {
            @Override
            public Amount apply(Invoice invoice) {
                return invoice.totalIncludingVAT();
            }
        }, invoicesOfTheMonth);
        return reduce(new BiFunction<Amount, Amount, Amount>() {
            @Override
            public Amount apply(Amount amount, Amount otherAmount) {
                return amount.add(otherAmount);
            }
        }, Amount.ZERO, includeVATAmounts);
    }

Il faut aussi définir les interfaces suivantes:

### Un prédicat

    public interface Predicate<T> {
        boolean test(T t);
    }

### Une fonction avec un seul paramètre

    public interface Function<T, R> {
        R apply(T t);
    }

### Une fonction avec deux paramètres

    public interface BiFunction<T,U,R> {
        R apply(T t, U u);
    }

L'écriture est moins concise qu'avec les `lambdas` et les `méthodes références`.

Pour exécuter cela et montrer comme çà marche nous allons implémenter de façon impérative les méthodes `reduce`, `map` et `filter`.

### `reduce`

    static <T, R> R reduce(BiFunction<R, T, R> reducer,
                           R identity, Collection<T> collection) {
        R accumulator = identity;
        for (T element: collection) {
            accumulator = reducer.apply(accumulator, element);
        }
        return accumulator;
    }

La fonction `reduce` prend en paramètre:

* La <i> fonction de réduction </i> (`reducer`);
* L'élément d'identité ou l'élément neutre (`identity`);
* Les éléments à réduire (`collection`).

Elle retourne le résultat après avoir appliqué la <i> fonction de réduction </i> autant de fois qu'il y a d'éléments à réduire.

### La fonction de réduction

    R reducer(R accumulator, T element)

C'est une fonction qui prend deux paramètres:

* Le résultat accumulé (`accumulator`);
* L'entrée à traiter (`element`).

Elle retourne le résultat accumulé (`result`) après traitement.

En java 8 nous pouvons écrire `reduce` de manière plus fonctionnelle avec l'API `Stream`:

    static <T, R> R reduce(BiFunction<R, T, R> reducer,
                           R identity,
                           BinaryOperator<R> combiner,
                           Collection<T> collection) {
        return collection.stream()
                         .reduce(identity, reducer, combiner);
    }


Il faut alors un paramètre supplémentaire requis (`combiner`) qui est utilisé pour reconstituer le résultat final si la fonction `reduce` s'est exécutée sur des `Stream` parallèles.

    static <T, R> R parallelReduce(BiFunction<R, T, R> reducer,
                                   R identity,
                                   BinaryOperator<R> combiner,
                                   Collection<T> collection) {
        return collection.parallelStream()
                         .reduce(identity, reducer, combiner);
    }

### `map`

    static <T, R> List<R> map(Function<T, R> mapper,
                              Collection<T> collection) {
        List<R> accumulator = new ArrayList<>();
        for (T element: collection) {
            accumulator.add(mapper.apply(element));
        }
        return accumulator;
    }

La fonction `map` prend en paramètre:

* Une fonction pour transformer tous les éléments fournis en entrée (`mapper`)
* L'ensemble des éléments à transformer (`collection`).

Elle retourne la liste des éléments transformés.

### La fonction de transformation

    R mapper(T element)

C'est une fonction qui prend en paramètre l'élément à transformer.
Elle retourne l'élément transformé.

En java 8 nous pouvons écrire `map` de manière plus fonctionnelle avec l'API `Stream`:

    static <T, R> List<R> map(Function<T, R> mapper,
                              Collection<T> collection) {
        return collection.stream()
                         .map(mapper).collect(Collectors.toList());
    }

### `filter`

    static <T> List<T> filter(Predicate<T> predicate,
                               Collection<T> collection) {
        List<T> accumulator = new ArrayList<>();
        for(T element: collection) {
            if (predicate.test(element)) {
                accumulator.add(element);
            }
        }
        return accumulator;
    }

La fonction `filter` prend en paramètre:

* Un prédicat qui est utilisé pour sélectionner les éléments (`predicate`).
* L'ensemble des éléments à sélectionner en fonction du prédicat (`collection`).

Elle retourne tous les éléments sélectionnés (ceux pour lesquels le prédicat est vrai).

### Le prédicat


    boolean predicate(T element)


C'est une fonction qui prend l'élément à tester en paramètre.
Elle retourne `true` si le prédicat est vrai pour l'élément, `false` sinon.

En java 8 nous pouvons écrire `filter` de manière plus fonctionnelle avec l'API `Stream`:


    static <T> List<T> filter(Predicate<T> predicate,
                              Collection<T> collection) {
        return collection.stream()
                         .filter(predicate)
                         .collect(Collectors.toList());
    }


### `reduce`peut accumuler des non-scalaires

Supposons que nous voulions compter le nombre d'occurence de chaque lettre dans la phrase <i> "Il était une fois une fonction reduce" </i>. Nous devrions obtenir le résultat suivant:


     Map<Character, Long> expectedResult = new HashMap<>();
        expectedResult.put(' ', 6L);
        expectedResult.put('I', 1L);
        expectedResult.put('a', 1L);
        expectedResult.put('c', 2L);
        expectedResult.put('d', 1L);
        expectedResult.put('e', 4L);
        expectedResult.put('f', 2L);
        expectedResult.put('i', 3L);
        expectedResult.put('l', 1L);
        expectedResult.put('n', 4L);
        expectedResult.put('o', 3L);
        expectedResult.put('r', 1L);
        expectedResult.put('s', 1L);
        expectedResult.put('t', 3L);
        expectedResult.put('u', 3L);
        expectedResult.put('é', 1L);

        assertThat(result).isEqualTo(expectedResult);


Pour construire le résultat attendu ici une `Map` nous pouvons utiliser `reduce`:


    List<Character> listOfCharacters =
         listOfCharacters("Il était une fois une fonction reduce");

    Map<Character, Long> result = reduce(accLetter,
                                         new HashMap<>(),
                                         listOfCharacters);


Nous transformons la phrase en liste de caractères comme suit:


    private List<Character> listOfCharacters(String s) {
        return s.chars()
                .mapToObj(c -> (char) c).collect(toList());
    }


La <i> fonction de réduction </i> (`accLetter`) est la suivante:


    final BiFunction<Map<Character, Long>, Character,
                     Map<Character, Long>> accLetter =
        (accumulator, letter) -> {
            accumulator.put(letter,
                       accumulator.getOrDefault(letter, 0L) + 1L);
            return accumulator;
        };


Elle prend en paramètres (`accumulator`) la `Map` contenant les compteurs de lettres dejà existant et la nouvelle lettre (`letter`) à traiter. Elle retourne la `Map` contenant les compteurs de lettres mis à jour.


    Map<Character, Long> accLetter(
          Map<Character, Long> letterCounters, Character letter)


## Troisième implémentation les `transducers`

Cette deuxième implémentation a les défauts suivants:

* L'ensemble des éléments est parcouru à chaque étape de transformation;
* Chaque étape de transformation crée une collection intermédiaire;
* Chaque étape supplémentaire augmente le temps de calcul et la mémoire consommée.

Dans la troisième implémentation nous allons procéder par étapes pour arriver au final aux `Transducers`

### Première étape: exprimons `map`et `filter` via `reduce`


    static <T, R> R reduce(BiFunction<R, T, R> reducer,
                           R identity,
                           BinaryOperator<R> combiner,
                           Collection<T> collection) {
        return collection.stream()
                         .reduce(identity, reducer, combiner);
    }


#### `map`avec `reduce`

Avant:


    static <T, R> List<R> map(Function<T, R> mapper,
                              Collection<T> collection) {
        List<R> accumulator = new ArrayList<>();
        for (T element: collection) {
            accumulator.add(mapper.apply(element));
        }
        return accumulator;
    }


Après:


    static <T, U, R extends Collection<U>> R map(
                   Function<T, U> mapper,
                   R result,
                   Collection<T> collection) {
        BiFunction<R, T, R> mapReducer =
           (accumulator, item) -> {
               accumulator.add(mapper.apply(item));
               return accumulator;
           };
        return reduce(mapReducer, result,
                      CollectionUtils::addAll, collection);
    }


Dans cette nouvelle version de `map` on remplace la boucle par un appel à la fonction `reduce`.
Pour appliquer cette fonction nous devons avoir:

* Une <i> fonction de réduction </i> (`mapReducer`) qui applique la transformation (`mapper`) à l'élément en entrée et ajoute l'élément transformé à la collection en cours de réduction (`result`).
* Une collection qui sera remplie par la réduction et qui contiendra les éléments transformés (`result`);

Le paramètre `combiner` de `reduce` est implémenté par la méthode `addAll` qui fusionne deux collections en une.


    static <T, R extends Collection<T>> R addAll(R c1, R c2) {
        c1.addAll(c2);
        return c1;
    }


#### `filter` avec `reduce`

Avant:


    static  <T> List<T> filter(Predicate<T> predicate,
                               Collection<T> collection) {
        List<T> accumulator = new ArrayList<>();
        for(T element: collection) {
            if (predicate.test(element)) {
                accumulator.add(element);
            }
        }
        return accumulator;
    }


Après:


    static <T, R extends Collection<T>> R filter(
                         Predicate<T> predicate,
                         R result,
                         Collection<T> collection) {
        BiFunction<R, T, R> filterReducer =
          (accumulator, item) -> {
            if (predicate.test(item)) {
                accumulator.add(item);
            }
            return accumulator;
          };
        return reduce(filterReducer, result,
                      CollectionUtils::addAll, collection);
    }


Dans cette nouvelle version de `filter` on remplace la boucle par un appel à la fonction `reduce`.
Pour appliquer cette fonction nous devons avoir:

* Une <i>fonction de réduction </i> (`filterReducer`) qui ajoute l'élément en entrée si il est sélectionné (`predicate`) à la collection en cours de réduction (`result`).
* Une collection qui sera remplie par la réduction et qui contiendra les éléments sélectionnés (`result`);

#### Implémentons `getAmountOf` avec les nouvelles fonctions `map` et `filter`

Avant:


    Amount getAmountOf(Month month) {
        List<Invoice> invoicesOfTheMonth =
            filter(invoice -> invoice.isDueFor(month), invoices());
        List<Amount> includeVATAmounts =
            map(Invoice::totalIncludingVAT, invoicesOfTheMonth);
        return reduce(Amount.ZERO, Amount::add, Amount::add,
                      includeVATAmounts);
    }


Après:


    Amount getAmountOf(Month month) {
        List<Invoice> invoicesOfTheMonth =
             filter(invoice -> invoice.isDueFor(month),
                    new ArrayList<>(), invoices());
        List<Amount> includeVATAmounts =
             map(Invoice::totalIncludingVAT,
                    new ArrayList<>(), invoicesOfTheMonth);
        return reduce(Amount::add, Amount.ZERO, Amount::add,
                      includeVATAmounts);
    }


Pas de gros changement si ce n'est la création des listes résultats lors des appels de `filter` et de `map`.

Au passage nous venons de voir comme construire `map`et `filter` à partir de `reduce`.

### Deuxième étape: sortons maintenant `reduce` des implémentations de `map` et `filter`

Lors de l'étape précédente le fait de construire `map` et `filter` avec `reduce` nous a permis de définir les <i> fonctions de réduction </i> qui permettent d'implémenter un `map` (`mapReducer`) et un `filter` (`filterReducer`).
Mais cette implémentation nous oblige aussi à avoir des collections intermédiaires.
Nous allons donc sortir `reduce` des implémentations de `map` et `filter`.

#### `map` devient uniquement une fabrique de <i> fonction de réduction </i>

Avant:


    static <T, U, R extends Collection<U>> R map(
                    Function<T, U> mapper,
                    R result,
                    Collection<T> collection) {
        BiFunction<R, T, R> mapReducer =
          (accumulator, item) -> {
            accumulator.add(mapper.apply(item));
            return accumulator;
          };
        return reduce(mapReducer, result,
                      CollectionUtils::addAll, collection);
    }


Après:


    static <T, U, R extends Collection<U>> BiFunction<R, T, R>
                                       map(Function<T, U> mapper) {
        return (accumulator, item) -> {
            accumulator.add(mapper.apply(item));
            return accumulator;
        };
    }


Dans cette version `map` devient une fabrique de <i> fonction de réduction </i> pour ajouter des éléments transformés dans la collection résultat.
Elle ne dépend plus des éléments à transformer, ni de la création de la collection résultat.
Elle a en paramètre uniquement la fonction de transformation (`mapper`).

#### `filter` devient uniquement une fabrique de <i> fonction de réduction </i>

Avant:


    static <T, R extends Collection<T>> R filter(
                                   Predicate<T> predicate,
                                   R result,
                                   Collection<T> collection) {
        BiFunction<R, T, R> filterReducer =
          (accumulator, item) -> {
            if (predicate.test(item)) {
                accumulator.add(item);
            }
            return accumulator;
          };
        return reduce(filterReducer, result,
                      CollectionUtils::addAll, collection);
    }


Après:


    static <T, R extends Collection<T>> BiFunction<R, T, R>
                                   filter(Predicate<T> predicate) {
        return (R accumulator, T item) -> {
            if (predicate.test(item)) {
                accumulator.add(item);
            }
            return accumulator;
        };
    }


Dans cette version `filter` devient une fabrique de <i> fonction de réduction </i> pour sélectionner les éléments correspondant au prédicat dans la collection résultat.
Elle ne dépend plus des éléments à sélectionner, ni de la création de la collection résultat.
Elle a en paramètre uniquement le prédicat à appliquer (`predicate`).

#### Implémentons `getAmountOf` avec les nouvelles fonctions `map` et `filter`

Avant:


    Amount getAmountOf(Month month) {
        List<Invoice> invoicesOfTheMonth =
               filter(invoice -> invoice.isDueFor(month),
                      new ArrayList<>(), invoices());
        List<Amount> includeVATAmounts =
               map(Invoice::totalIncludingVAT,
                     new ArrayList<>(), invoicesOfTheMonth);
        return reduce(Amount::add, Amount.ZERO, Amount::add,
                      includeVATAmounts);
    }


Après:


    Amount getAmountOf(Month month) {
        List<Invoice> invoicesOfTheMonth =
              reduce(filter(invoice -> invoice.isDueFor(month)),
                            new ArrayList<>(),
                            CollectionUtils::addAll, invoices());
        List<Amount> includeVATAmounts =
             reduce(map(Invoice::totalIncludingVAT),
                        new ArrayList<>(),
                        CollectionUtils::addAll,
                        invoicesOfTheMonth);
        return reduce(Amount::add, Amount.ZERO, Amount::add,
                      includeVATAmounts);
    }


Maintenant que nous avons sorti `reduce` de `map` et `filter` nous allons supprimer les `reduce` intermédiaires et donc les collections intermédiaires.

### Troisième étape: supprimons les listes intermédiaires

Nous souhaitons obtenir l'implémentation de `getAmountOf` suivante:


    Amount getAmountOf(Month month) {
        return reduce(Amount.ZERO,
                (filter((Invoice invoice) ->
                                invoice.isDueFor(month))
                     .andThen(map(Invoice::totalIncludingVAT)))
                     .apply(Amount::add),
                Amount::add, invoices());
    }


Pour rappel la fonction `reduce` prend en paramètre une <i> fonction de réduction </i> qui a la signature suivante:


    R reducer(R accumulator, T element)


Il faut donc que la fonction créée par l'application de `Amount::add` à la composition de fonction soit du même type que `reducer`.

Dans notre exemple on a `Amount::add` qui est une fonction avec la signature suivante:


    BiFonction<Amount, Amount, Amount>


Pour réduire une collection de factures (`Invoice`) en montant (`Amount`) il faut une fonction avec la signature suivante:


    BiFonction<Amount, Invoice, Amount>


Il faut donc que la méthode `apply` ait la signature suivante:


    BiFonction<Amount, Invoice, Amount> apply(
                  BiFunction<Amount, Amount, Amount> nextReducer)


Si on généralise on obtient la signature suivante:


    BiFunction<R, U, R> apply(BiFunction<R, T, R> nextReducer)


Avec:


    R <=> Amount
    U <=> Invoice
    T <=> Amount


La fonction `apply` permet de créer une chaîne de <i> fonctions de réduction </i>, c'est ce qu'on appel un <b>`Transducer`</b>.


    abstract class Transducer<T, U> {

        private Transducer() {}

        abstract <R> BiFunction<R, U, R> apply(
                                 BiFunction<R, T, R> nextReducer);

        ...
    }


Pour sélectionner les factures à payer il faut une fonction `filter` qui fabrique un `Transducer`:


    filter((Invoice invoice) -> invoice.isDueFor(month))

    static <V> Transducer<V, V> filter(Predicate<V> predicate) {
        Objects.requireNonNull(predicate);
        return new Transducer<V, V>() {
            @Override
            public <R> BiFunction<R, V, R> apply(
                                BiFunction<R, V, R> nextReducer) {
                return (accumulator, input) -> {
                    if (predicate.test(input)) {
                        return nextReducer
                                   .apply(accumulator, input);
                    }
                    return accumulator;
                };
            }
        };
    }


Quand la <i> fonction de réduction </i> est appliquée, le prédicat est testé et si l'élément courant est sélectionné, la <i> fonction de réduction </i> suivante est appliquée avec l'accumulateur en cours (`accumulator`) et l'élément sélectionné (`input`) sinon l'accumulateur (`accumulator`) est retourné.

Pour transformer les factures à payer en montant toutes taxes comprises, il faut une fonction `map` qui créer un <b>`Transducer`</b>.


    map(Invoice::totalIncludingVAT)

    static <T, U> Transducer<T, U> map(Function<U, T> mapper) {
        Objects.requireNonNull(mapper);
        return new Transducer<T, U>() {
            @Override
            public <R> BiFunction<R, U, R> apply(
                                BiFunction<R, T, R> nextReducer) {
                return (accumulator, input) -> nextReducer
                        .apply(accumulator, mapper.apply(input));
            }
        };
    }


Quand la <i> fonction de réduction </i> est appliquée, la transformation (`mapper`) est appliquée sur l'élément à transformer et la <i> fonction de réduction </i> suivante est appliquée avec l'accumulateur en cours (`accumulator`) et l'élément transformé (`mapper.apply(input)`).

Pour pouvoir composer une fonction à partir des fonctions comme `filter` et `map` on ajoute la méthode `andThen` qui fabrique un nouveau `Transducer`.


    nextReducer -> filter.apply(mapper.apply(nextReducer)

    <V> Transducer<V, U> andThen(Transducer<V, T> after) {
        Objects.requireNonNull(after);
        return new Transducer<V, U>() {
            @Override
            public <R> BiFunction<R, U, R> apply(
                               BiFunction<R, V, R> nextReducer) {
                return Transducer.this.apply(
                                   after.apply(nextReducer));
            }
        };
    }


Quand la <i> fonction de réduction </i> est appliquée le `Transducer` `mapper` est créé en appliquant `after.apply(nextReducer)`, puis le `Transducer` `filter` est créé en appliquant le `Transducer` courant (`Transducer.this.apply(...)` avec le `Transducer` `mapper` en paramètre (créé par `after.apply(nextReducer)`).

On peut maintenant définir tous les `Transducers` que l'on souhaite:


    Transducer<Invoice, Invoice> t1 = filter(
                  (Invoice invoice) -> invoice.isDueFor(month));

    Transducer<Amount, Invoice> t2 = map(
                  Invoice::totalIncludingVAT);

    Transducer<Amount, Invoice> t3 = t1.andThen(t2);


Mais ce ne sont que des <i> définitions de fonction </i>. Elles créent une <i> fonction de réduction </i> que lorsque nous les appliquons avec en paramètre une autre <i> fonction de réduction </i>.

Par exemple si on veut la liste des factures dues:


    Transducer<Invoice, Invoice> t1 = filter(
            (Invoice invoice) -> invoice.isDueFor(Month.OCTOBER));

    BiFunction<List<Invoice>, Invoice, List<Invoice>> reducer =
            t1.apply(CollectionUtils::add);

    assertThat(reduce(reducer, new ArrayList<>(),
            CollectionUtils::addAll, invoices()))
       .extracting(Invoice::totalIncludingVAT)
       .containsOnly(Amount.of(43.20), Amount.of(35.70));


On voit ici que si on crée un `Transducer` (`t1`) pour sélectionner les factures dues, nous pouvons créer une <i> fonction de réduction </i> (`reducer`). Si nous appelons, `reduce` avec la fonction `reducer`, nous obtenons la liste des factures dues.

Nous venons donc de voir que les `Transducers` sont des <i>définitions de fonction </i> qui peuvent être utilisées seules ou composées de plusieurs `Transducers`. Nous pouvons appliquer n'importe quelle <i> fonction de réduction </i> au `Transducer` pour créer un `reducer`. Cette technique permet de réutiliser les `Transducers` et de les tester séparément. Un `Reducer` peut être passé en paramètre à n'importe quelle implémentation de `reduce` ou équivalent et les éléments à réduire peuvent provenir aussi bien d'une collection que de flux d'une socket, d'un fichier, etc... Cela permet de marier les `Transducers` avec la programmation réactive par exemple, mais ceci peut faire l'objet d'un article à part entière.

Si vous souhaitez retrouver les sources de cet article, ils sont disponibles sur mon [`Github`](https://github.com/PatrickGIRY/transducers).
