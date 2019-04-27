
# Strings {#strings .r4ds-section}

## Introduction {#introduction-8 .r4ds-section}


```r
library(tidyverse)
library(stringr)
```

## String basics {#string-basics .r4ds-section}

### Exercise 14.2.1 {.unnumbered .exercise data-number="14.2.1"}

<div class="question">

In code that doesn’t use stringr, you’ll often see `paste()` and `paste0()`. 
What’s the difference between the two functions? What stringr function are they equivalent to? 
How do the functions differ in their handling of `NA`?

</div>

<div class="answer">

The function `paste()` separates strings by spaces by default, while `paste0()` does not separate strings with spaces by default.


```r
paste("foo", "bar")
#> [1] "foo bar"
paste0("foo", "bar")
#> [1] "foobar"
```

Since `str_c()` does not separate strings with spaces by default it is closer in behavior to `paste0()`.


```r
str_c("foo", "bar")
#> [1] "foobar"
```

However, `str_c()` and the paste function handle NA differently.
The function `str_c()` propagates `NA`, if any argument is a missing value, it returns a missing value.
This is in line with how the numeric R functions, e.g. `sum()`, `mean()`, handle missing values.
However, the paste functions, convert `NA` to the string `"NA"` and then treat it as any other character vector.

```r
str_c("foo", NA)
#> [1] NA
paste("foo", NA)
#> [1] "foo NA"
paste0("foo", NA)
#> [1] "fooNA"
```

</div>

### Exercise 14.2.2 {.unnumbered .exercise data-number="14.2.2"}

<div class="question">
In your own words, describe the difference between the `sep` and `collapse` arguments to `str_c()`.
</div>

<div class="answer">

The `sep` argument is the string inserted between arguments to `str_c()`, while `collapse` is the string used to separate any elements of the character vector into a character vector of length one.

</div>

### Exercise 14.2.3 {.unnumbered .exercise data-number="14.2.3"}

<div class="question">
Use `str_length()` and `str_sub()` to extract the middle character from a string. What will you do if the string has an even number of characters?
</div>

<div class="answer">

The following function extracts the middle character. If the string has an even number of characters the choice is arbitrary.
We choose to select $\lceil n / 2 \rceil$, because that case works even if the string is only of length one.
A more general method would allow the user to select either the floor or ceiling for the middle character of an even string.

```r
x <- c("a", "abc", "abcd", "abcde", "abcdef")
L <- str_length(x)
m <- ceiling(L / 2)
str_sub(x, m, m)
#> [1] "a" "b" "b" "c" "c"
```

</div>

### Exercise 14.2.4 {.unnumbered .exercise data-number="14.2.4"}

<div class="question">
What does `str_wrap()` do? When might you want to use it?
</div>

<div class="answer">

The function `str_wrap()` wraps text so that it fits within a certain width.
This is useful for wrapping long strings of text to be typeset.

</div>

### Exercise 14.2.5 {.unnumbered .exercise data-number="14.2.5"}

<div class="question">
What does `str_trim()` do? What’s the opposite of `str_trim()`?
</div>

<div class="answer">

The function `str_trim()` trims the whitespace from a string.

```r
str_trim(" abc ")
#> [1] "abc"
str_trim(" abc ", side = "left")
#> [1] "abc "
str_trim(" abc ", side = "right")
#> [1] " abc"
```

The opposite of `str_trim()` is `str_pad()` which adds characters to each side.


```r
str_pad("abc", 5, side = "both")
#> [1] " abc "
str_pad("abc", 4, side = "right")
#> [1] "abc "
str_pad("abc", 4, side = "left")
#> [1] " abc"
```

</div>

### Exercise 14.2.6 {.unnumbered .exercise data-number="14.2.6"}

<div class="question">
Write a function that turns (e.g.) a vector `c("a", "b", "c")` into the string `"a, b, and c"`. Think carefully about what it should do if given a vector of length 0, 1, or 2.
</div>

<div class="answer">

See the Chapter [Functions] for more details on writing R functions.

This function needs to handle four cases.

1.  `n == 0`: an empty string, e.g. `""`.
1.  `n == 1`: the original vector, e.g. `"a"`.
1.  `n == 2`: return the two elements separated by "and", e.g. `"a and b"`.
1.  `n > 2`: return the first `n - 1` elements separated by commas, and the last element separated by a comma and "and", e.g. `"a, b, and c"`.


```r
str_commasep <- function(x, delim = ",") {
  n <- length(x)
  if (n == 0) {
    ""
  } else if (n == 1) {
    x
  } else if (n == 2) {
    # no comma before and when n == 2
    str_c(x[[1]], "and", x[[2]], sep = " ")
  } else {
    # commas after all n - 1 elements
    not_last <- str_c(x[seq_len(n - 1)], delim)
    # prepend "and" to the last element
    last <- str_c("and", x[[n]], sep = " ")
    # combine parts with spaces
    str_c(c(not_last, last), collapse = " ")
  }
}
str_commasep("")
#> [1] ""
str_commasep("a")
#> [1] "a"
str_commasep(c("a", "b"))
#> [1] "a and b"
str_commasep(c("a", "b", "c"))
#> [1] "a, b, and c"
str_commasep(c("a", "b", "c", "d"))
#> [1] "a, b, c, and d"
```

</div>

## Matching patterns with regular expressions {#matching-patterns-with-regular-expressions .r4ds-section}

### Basic matches {#basic-matches .r4ds-section}

#### Exercise 14.3.1.1 {.unnumbered .exercise data-number="14.3.1.1"}

<div class="question">
Explain why each of these strings don’t match a `\`: `"\"`, `"\\"`, `"\\\"`.
</div>

<div class="answer">

-   `"\"`: This will escape the next character in the R string.
-   `"\\"`: This will resolve to `\` in the regular expression, which will escape the next character in the regular expression.
-   `"\\\"`: The first two backslashes will resolve to a literal backslash in the regular expression, the third will escape the next character. So in the regular expression, this will escape some escaped character.

</div>

#### Exercise 14.3.1.2 {.unnumbered .exercise data-number="14.3.1.2"}

<div class="question">
How would you match the sequence `"'\` ?
</div>

<div class="answer">


```r
str_view("\"'\\", "\"'\\\\", match = TRUE)
```

<!--html_preserve--><div id="htmlwidget-14d5992801777f4abbc5" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-14d5992801777f4abbc5">{"x":{"html":"<ul>\n  <li><span class='match'>\"'\\<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

</div>

#### Exercise 14.3.1.3 {.unnumbered .exercise data-number="14.3.1.3"}

<div class="question">
What patterns will the regular expression `\..\..\..` match? How would you represent it as a string?
</div>

<div class="answer">

It will match any patterns that are a dot followed by any character, repeated three times.


```r
str_view(c(".a.b.c", ".a.b", "....."), c("\\..\\..\\.."), match = TRUE)
```

<!--html_preserve--><div id="htmlwidget-df2c08526632671063f9" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-df2c08526632671063f9">{"x":{"html":"<ul>\n  <li><span class='match'>.a.b.c<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

</div>

### Anchors {#anchors .r4ds-section}

#### Exercise 14.3.2.1 {.unnumbered .exercise data-number="14.3.2.1"}

<div class="question">
How would you match the literal string `"$^$"`?
</div>

<div class="answer">


```r
str_view(c("$^$", "ab$^$sfas"), "^\\$\\^\\$$", match = TRUE)
```

<!--html_preserve--><div id="htmlwidget-4aadbc32fbbd0d87b2b0" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-4aadbc32fbbd0d87b2b0">{"x":{"html":"<ul>\n  <li><span class='match'>$^$<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

</div>

#### Exercise 14.3.2.2 {.unnumbered .exercise data-number="14.3.2.2"}

<div class="question">
Given the corpus of common words in `stringr::words`, create regular expressions that find all words that:

1.  Start with “y”.
1.  End with “x”
1.  Are exactly three letters long. (Don’t cheat by using `str_length()`!)
1.  Have seven letters or more.

Since this list is long, you might want to use the `match` argument to `str_view()` to show only the matching or non-matching words.

</div>

<div class="answer">

The answer to each part follows.

1.  The words that start with  “y” are:

    
    ```r
    str_view(stringr::words, "^y", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-07394da27a6eb4f22e37" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-07394da27a6eb4f22e37">{"x":{"html":"<ul>\n  <li><span class='match'>y<\/span>ear<\/li>\n  <li><span class='match'>y<\/span>es<\/li>\n  <li><span class='match'>y<\/span>esterday<\/li>\n  <li><span class='match'>y<\/span>et<\/li>\n  <li><span class='match'>y<\/span>ou<\/li>\n  <li><span class='match'>y<\/span>oung<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

1.  End with “x”

    
    ```r
    str_view(stringr::words, "x$", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-ae7fa4a918c3c5fdf863" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-ae7fa4a918c3c5fdf863">{"x":{"html":"<ul>\n  <li>bo<span class='match'>x<\/span><\/li>\n  <li>se<span class='match'>x<\/span><\/li>\n  <li>si<span class='match'>x<\/span><\/li>\n  <li>ta<span class='match'>x<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

1.  Are exactly three letters long are

    
    ```r
    str_view(stringr::words, "^...$", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-76502c887ec734b6105a" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-76502c887ec734b6105a">{"x":{"html":"<ul>\n  <li><span class='match'>act<\/span><\/li>\n  <li><span class='match'>add<\/span><\/li>\n  <li><span class='match'>age<\/span><\/li>\n  <li><span class='match'>ago<\/span><\/li>\n  <li><span class='match'>air<\/span><\/li>\n  <li><span class='match'>all<\/span><\/li>\n  <li><span class='match'>and<\/span><\/li>\n  <li><span class='match'>any<\/span><\/li>\n  <li><span class='match'>arm<\/span><\/li>\n  <li><span class='match'>art<\/span><\/li>\n  <li><span class='match'>ask<\/span><\/li>\n  <li><span class='match'>bad<\/span><\/li>\n  <li><span class='match'>bag<\/span><\/li>\n  <li><span class='match'>bar<\/span><\/li>\n  <li><span class='match'>bed<\/span><\/li>\n  <li><span class='match'>bet<\/span><\/li>\n  <li><span class='match'>big<\/span><\/li>\n  <li><span class='match'>bit<\/span><\/li>\n  <li><span class='match'>box<\/span><\/li>\n  <li><span class='match'>boy<\/span><\/li>\n  <li><span class='match'>bus<\/span><\/li>\n  <li><span class='match'>but<\/span><\/li>\n  <li><span class='match'>buy<\/span><\/li>\n  <li><span class='match'>can<\/span><\/li>\n  <li><span class='match'>car<\/span><\/li>\n  <li><span class='match'>cat<\/span><\/li>\n  <li><span class='match'>cup<\/span><\/li>\n  <li><span class='match'>cut<\/span><\/li>\n  <li><span class='match'>dad<\/span><\/li>\n  <li><span class='match'>day<\/span><\/li>\n  <li><span class='match'>die<\/span><\/li>\n  <li><span class='match'>dog<\/span><\/li>\n  <li><span class='match'>dry<\/span><\/li>\n  <li><span class='match'>due<\/span><\/li>\n  <li><span class='match'>eat<\/span><\/li>\n  <li><span class='match'>egg<\/span><\/li>\n  <li><span class='match'>end<\/span><\/li>\n  <li><span class='match'>eye<\/span><\/li>\n  <li><span class='match'>far<\/span><\/li>\n  <li><span class='match'>few<\/span><\/li>\n  <li><span class='match'>fit<\/span><\/li>\n  <li><span class='match'>fly<\/span><\/li>\n  <li><span class='match'>for<\/span><\/li>\n  <li><span class='match'>fun<\/span><\/li>\n  <li><span class='match'>gas<\/span><\/li>\n  <li><span class='match'>get<\/span><\/li>\n  <li><span class='match'>god<\/span><\/li>\n  <li><span class='match'>guy<\/span><\/li>\n  <li><span class='match'>hit<\/span><\/li>\n  <li><span class='match'>hot<\/span><\/li>\n  <li><span class='match'>how<\/span><\/li>\n  <li><span class='match'>job<\/span><\/li>\n  <li><span class='match'>key<\/span><\/li>\n  <li><span class='match'>kid<\/span><\/li>\n  <li><span class='match'>lad<\/span><\/li>\n  <li><span class='match'>law<\/span><\/li>\n  <li><span class='match'>lay<\/span><\/li>\n  <li><span class='match'>leg<\/span><\/li>\n  <li><span class='match'>let<\/span><\/li>\n  <li><span class='match'>lie<\/span><\/li>\n  <li><span class='match'>lot<\/span><\/li>\n  <li><span class='match'>low<\/span><\/li>\n  <li><span class='match'>man<\/span><\/li>\n  <li><span class='match'>may<\/span><\/li>\n  <li><span class='match'>mrs<\/span><\/li>\n  <li><span class='match'>new<\/span><\/li>\n  <li><span class='match'>non<\/span><\/li>\n  <li><span class='match'>not<\/span><\/li>\n  <li><span class='match'>now<\/span><\/li>\n  <li><span class='match'>odd<\/span><\/li>\n  <li><span class='match'>off<\/span><\/li>\n  <li><span class='match'>old<\/span><\/li>\n  <li><span class='match'>one<\/span><\/li>\n  <li><span class='match'>out<\/span><\/li>\n  <li><span class='match'>own<\/span><\/li>\n  <li><span class='match'>pay<\/span><\/li>\n  <li><span class='match'>per<\/span><\/li>\n  <li><span class='match'>put<\/span><\/li>\n  <li><span class='match'>red<\/span><\/li>\n  <li><span class='match'>rid<\/span><\/li>\n  <li><span class='match'>run<\/span><\/li>\n  <li><span class='match'>say<\/span><\/li>\n  <li><span class='match'>see<\/span><\/li>\n  <li><span class='match'>set<\/span><\/li>\n  <li><span class='match'>sex<\/span><\/li>\n  <li><span class='match'>she<\/span><\/li>\n  <li><span class='match'>sir<\/span><\/li>\n  <li><span class='match'>sit<\/span><\/li>\n  <li><span class='match'>six<\/span><\/li>\n  <li><span class='match'>son<\/span><\/li>\n  <li><span class='match'>sun<\/span><\/li>\n  <li><span class='match'>tax<\/span><\/li>\n  <li><span class='match'>tea<\/span><\/li>\n  <li><span class='match'>ten<\/span><\/li>\n  <li><span class='match'>the<\/span><\/li>\n  <li><span class='match'>tie<\/span><\/li>\n  <li><span class='match'>too<\/span><\/li>\n  <li><span class='match'>top<\/span><\/li>\n  <li><span class='match'>try<\/span><\/li>\n  <li><span class='match'>two<\/span><\/li>\n  <li><span class='match'>use<\/span><\/li>\n  <li><span class='match'>war<\/span><\/li>\n  <li><span class='match'>way<\/span><\/li>\n  <li><span class='match'>wee<\/span><\/li>\n  <li><span class='match'>who<\/span><\/li>\n  <li><span class='match'>why<\/span><\/li>\n  <li><span class='match'>win<\/span><\/li>\n  <li><span class='match'>yes<\/span><\/li>\n  <li><span class='match'>yet<\/span><\/li>\n  <li><span class='match'>you<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

1.  The words that have seven letters or more are

    
    ```r
    str_view(stringr::words, ".......", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-d34691559831f28a8b47" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-d34691559831f28a8b47">{"x":{"html":"<ul>\n  <li><span class='match'>absolut<\/span>e<\/li>\n  <li><span class='match'>account<\/span><\/li>\n  <li><span class='match'>achieve<\/span><\/li>\n  <li><span class='match'>address<\/span><\/li>\n  <li><span class='match'>adverti<\/span>se<\/li>\n  <li><span class='match'>afterno<\/span>on<\/li>\n  <li><span class='match'>against<\/span><\/li>\n  <li><span class='match'>already<\/span><\/li>\n  <li><span class='match'>alright<\/span><\/li>\n  <li><span class='match'>althoug<\/span>h<\/li>\n  <li><span class='match'>america<\/span><\/li>\n  <li><span class='match'>another<\/span><\/li>\n  <li><span class='match'>apparen<\/span>t<\/li>\n  <li><span class='match'>appoint<\/span><\/li>\n  <li><span class='match'>approac<\/span>h<\/li>\n  <li><span class='match'>appropr<\/span>iate<\/li>\n  <li><span class='match'>arrange<\/span><\/li>\n  <li><span class='match'>associa<\/span>te<\/li>\n  <li><span class='match'>authori<\/span>ty<\/li>\n  <li><span class='match'>availab<\/span>le<\/li>\n  <li><span class='match'>balance<\/span><\/li>\n  <li><span class='match'>because<\/span><\/li>\n  <li><span class='match'>believe<\/span><\/li>\n  <li><span class='match'>benefit<\/span><\/li>\n  <li><span class='match'>between<\/span><\/li>\n  <li><span class='match'>brillia<\/span>nt<\/li>\n  <li><span class='match'>britain<\/span><\/li>\n  <li><span class='match'>brother<\/span><\/li>\n  <li><span class='match'>busines<\/span>s<\/li>\n  <li><span class='match'>certain<\/span><\/li>\n  <li><span class='match'>chairma<\/span>n<\/li>\n  <li><span class='match'>charact<\/span>er<\/li>\n  <li><span class='match'>Christm<\/span>as<\/li>\n  <li><span class='match'>colleag<\/span>ue<\/li>\n  <li><span class='match'>collect<\/span><\/li>\n  <li><span class='match'>college<\/span><\/li>\n  <li><span class='match'>comment<\/span><\/li>\n  <li><span class='match'>committ<\/span>ee<\/li>\n  <li><span class='match'>communi<\/span>ty<\/li>\n  <li><span class='match'>company<\/span><\/li>\n  <li><span class='match'>compare<\/span><\/li>\n  <li><span class='match'>complet<\/span>e<\/li>\n  <li><span class='match'>compute<\/span><\/li>\n  <li><span class='match'>concern<\/span><\/li>\n  <li><span class='match'>conditi<\/span>on<\/li>\n  <li><span class='match'>conside<\/span>r<\/li>\n  <li><span class='match'>consult<\/span><\/li>\n  <li><span class='match'>contact<\/span><\/li>\n  <li><span class='match'>continu<\/span>e<\/li>\n  <li><span class='match'>contrac<\/span>t<\/li>\n  <li><span class='match'>control<\/span><\/li>\n  <li><span class='match'>convers<\/span>e<\/li>\n  <li><span class='match'>correct<\/span><\/li>\n  <li><span class='match'>council<\/span><\/li>\n  <li><span class='match'>country<\/span><\/li>\n  <li><span class='match'>current<\/span><\/li>\n  <li><span class='match'>decisio<\/span>n<\/li>\n  <li><span class='match'>definit<\/span>e<\/li>\n  <li><span class='match'>departm<\/span>ent<\/li>\n  <li><span class='match'>describ<\/span>e<\/li>\n  <li><span class='match'>develop<\/span><\/li>\n  <li><span class='match'>differe<\/span>nce<\/li>\n  <li><span class='match'>difficu<\/span>lt<\/li>\n  <li><span class='match'>discuss<\/span><\/li>\n  <li><span class='match'>distric<\/span>t<\/li>\n  <li><span class='match'>documen<\/span>t<\/li>\n  <li><span class='match'>economy<\/span><\/li>\n  <li><span class='match'>educate<\/span><\/li>\n  <li><span class='match'>electri<\/span>c<\/li>\n  <li><span class='match'>encoura<\/span>ge<\/li>\n  <li><span class='match'>english<\/span><\/li>\n  <li><span class='match'>environ<\/span>ment<\/li>\n  <li><span class='match'>especia<\/span>l<\/li>\n  <li><span class='match'>evening<\/span><\/li>\n  <li><span class='match'>evidenc<\/span>e<\/li>\n  <li><span class='match'>example<\/span><\/li>\n  <li><span class='match'>exercis<\/span>e<\/li>\n  <li><span class='match'>expense<\/span><\/li>\n  <li><span class='match'>experie<\/span>nce<\/li>\n  <li><span class='match'>explain<\/span><\/li>\n  <li><span class='match'>express<\/span><\/li>\n  <li><span class='match'>finance<\/span><\/li>\n  <li><span class='match'>fortune<\/span><\/li>\n  <li><span class='match'>forward<\/span><\/li>\n  <li><span class='match'>functio<\/span>n<\/li>\n  <li><span class='match'>further<\/span><\/li>\n  <li><span class='match'>general<\/span><\/li>\n  <li><span class='match'>germany<\/span><\/li>\n  <li><span class='match'>goodbye<\/span><\/li>\n  <li><span class='match'>history<\/span><\/li>\n  <li><span class='match'>holiday<\/span><\/li>\n  <li><span class='match'>hospita<\/span>l<\/li>\n  <li><span class='match'>however<\/span><\/li>\n  <li><span class='match'>hundred<\/span><\/li>\n  <li><span class='match'>husband<\/span><\/li>\n  <li><span class='match'>identif<\/span>y<\/li>\n  <li><span class='match'>imagine<\/span><\/li>\n  <li><span class='match'>importa<\/span>nt<\/li>\n  <li><span class='match'>improve<\/span><\/li>\n  <li><span class='match'>include<\/span><\/li>\n  <li><span class='match'>increas<\/span>e<\/li>\n  <li><span class='match'>individ<\/span>ual<\/li>\n  <li><span class='match'>industr<\/span>y<\/li>\n  <li><span class='match'>instead<\/span><\/li>\n  <li><span class='match'>interes<\/span>t<\/li>\n  <li><span class='match'>introdu<\/span>ce<\/li>\n  <li><span class='match'>involve<\/span><\/li>\n  <li><span class='match'>kitchen<\/span><\/li>\n  <li><span class='match'>languag<\/span>e<\/li>\n  <li><span class='match'>machine<\/span><\/li>\n  <li><span class='match'>meaning<\/span><\/li>\n  <li><span class='match'>measure<\/span><\/li>\n  <li><span class='match'>mention<\/span><\/li>\n  <li><span class='match'>million<\/span><\/li>\n  <li><span class='match'>ministe<\/span>r<\/li>\n  <li><span class='match'>morning<\/span><\/li>\n  <li><span class='match'>necessa<\/span>ry<\/li>\n  <li><span class='match'>obvious<\/span><\/li>\n  <li><span class='match'>occasio<\/span>n<\/li>\n  <li><span class='match'>operate<\/span><\/li>\n  <li><span class='match'>opportu<\/span>nity<\/li>\n  <li><span class='match'>organiz<\/span>e<\/li>\n  <li><span class='match'>origina<\/span>l<\/li>\n  <li><span class='match'>otherwi<\/span>se<\/li>\n  <li><span class='match'>paragra<\/span>ph<\/li>\n  <li><span class='match'>particu<\/span>lar<\/li>\n  <li><span class='match'>pension<\/span><\/li>\n  <li><span class='match'>percent<\/span><\/li>\n  <li><span class='match'>perfect<\/span><\/li>\n  <li><span class='match'>perhaps<\/span><\/li>\n  <li><span class='match'>photogr<\/span>aph<\/li>\n  <li><span class='match'>picture<\/span><\/li>\n  <li><span class='match'>politic<\/span><\/li>\n  <li><span class='match'>positio<\/span>n<\/li>\n  <li><span class='match'>positiv<\/span>e<\/li>\n  <li><span class='match'>possibl<\/span>e<\/li>\n  <li><span class='match'>practis<\/span>e<\/li>\n  <li><span class='match'>prepare<\/span><\/li>\n  <li><span class='match'>present<\/span><\/li>\n  <li><span class='match'>pressur<\/span>e<\/li>\n  <li><span class='match'>presume<\/span><\/li>\n  <li><span class='match'>previou<\/span>s<\/li>\n  <li><span class='match'>private<\/span><\/li>\n  <li><span class='match'>probabl<\/span>e<\/li>\n  <li><span class='match'>problem<\/span><\/li>\n  <li><span class='match'>proceed<\/span><\/li>\n  <li><span class='match'>process<\/span><\/li>\n  <li><span class='match'>produce<\/span><\/li>\n  <li><span class='match'>product<\/span><\/li>\n  <li><span class='match'>program<\/span>me<\/li>\n  <li><span class='match'>project<\/span><\/li>\n  <li><span class='match'>propose<\/span><\/li>\n  <li><span class='match'>protect<\/span><\/li>\n  <li><span class='match'>provide<\/span><\/li>\n  <li><span class='match'>purpose<\/span><\/li>\n  <li><span class='match'>quality<\/span><\/li>\n  <li><span class='match'>quarter<\/span><\/li>\n  <li><span class='match'>questio<\/span>n<\/li>\n  <li><span class='match'>realise<\/span><\/li>\n  <li><span class='match'>receive<\/span><\/li>\n  <li><span class='match'>recogni<\/span>ze<\/li>\n  <li><span class='match'>recomme<\/span>nd<\/li>\n  <li><span class='match'>relatio<\/span>n<\/li>\n  <li><span class='match'>remembe<\/span>r<\/li>\n  <li><span class='match'>represe<\/span>nt<\/li>\n  <li><span class='match'>require<\/span><\/li>\n  <li><span class='match'>researc<\/span>h<\/li>\n  <li><span class='match'>resourc<\/span>e<\/li>\n  <li><span class='match'>respect<\/span><\/li>\n  <li><span class='match'>respons<\/span>ible<\/li>\n  <li><span class='match'>saturda<\/span>y<\/li>\n  <li><span class='match'>science<\/span><\/li>\n  <li><span class='match'>scotlan<\/span>d<\/li>\n  <li><span class='match'>secreta<\/span>ry<\/li>\n  <li><span class='match'>section<\/span><\/li>\n  <li><span class='match'>separat<\/span>e<\/li>\n  <li><span class='match'>serious<\/span><\/li>\n  <li><span class='match'>service<\/span><\/li>\n  <li><span class='match'>similar<\/span><\/li>\n  <li><span class='match'>situate<\/span><\/li>\n  <li><span class='match'>society<\/span><\/li>\n  <li><span class='match'>special<\/span><\/li>\n  <li><span class='match'>specifi<\/span>c<\/li>\n  <li><span class='match'>standar<\/span>d<\/li>\n  <li><span class='match'>station<\/span><\/li>\n  <li><span class='match'>straigh<\/span>t<\/li>\n  <li><span class='match'>strateg<\/span>y<\/li>\n  <li><span class='match'>structu<\/span>re<\/li>\n  <li><span class='match'>student<\/span><\/li>\n  <li><span class='match'>subject<\/span><\/li>\n  <li><span class='match'>succeed<\/span><\/li>\n  <li><span class='match'>suggest<\/span><\/li>\n  <li><span class='match'>support<\/span><\/li>\n  <li><span class='match'>suppose<\/span><\/li>\n  <li><span class='match'>surpris<\/span>e<\/li>\n  <li><span class='match'>telepho<\/span>ne<\/li>\n  <li><span class='match'>televis<\/span>ion<\/li>\n  <li><span class='match'>terribl<\/span>e<\/li>\n  <li><span class='match'>therefo<\/span>re<\/li>\n  <li><span class='match'>thirtee<\/span>n<\/li>\n  <li><span class='match'>thousan<\/span>d<\/li>\n  <li><span class='match'>through<\/span><\/li>\n  <li><span class='match'>thursda<\/span>y<\/li>\n  <li><span class='match'>togethe<\/span>r<\/li>\n  <li><span class='match'>tomorro<\/span>w<\/li>\n  <li><span class='match'>tonight<\/span><\/li>\n  <li><span class='match'>traffic<\/span><\/li>\n  <li><span class='match'>transpo<\/span>rt<\/li>\n  <li><span class='match'>trouble<\/span><\/li>\n  <li><span class='match'>tuesday<\/span><\/li>\n  <li><span class='match'>underst<\/span>and<\/li>\n  <li><span class='match'>univers<\/span>ity<\/li>\n  <li><span class='match'>various<\/span><\/li>\n  <li><span class='match'>village<\/span><\/li>\n  <li><span class='match'>wednesd<\/span>ay<\/li>\n  <li><span class='match'>welcome<\/span><\/li>\n  <li><span class='match'>whether<\/span><\/li>\n  <li><span class='match'>without<\/span><\/li>\n  <li><span class='match'>yesterd<\/span>ay<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

</div>

### Character classes and alternatives {#character-classes-and-alternatives .r4ds-section}

#### Exercise 14.3.3.1 {.unnumbered .exercise data-number="14.3.3.1"}

<div class="question">

Create regular expressions to find all words that:

1.  Start with a vowel.
1.  That only contain consonants. (Hint: thinking about matching “not”-vowels.)
1.  End with `ed`, but not with `eed`.
1.  End with `ing` or `ise`.

</div>

<div class="answer">

The answer to each part follows.

1.  Words starting with vowels

    
    ```r
    str_view(stringr::words, "^[aeiou]", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-725f07776305608fdb62" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-725f07776305608fdb62">{"x":{"html":"<ul>\n  <li><span class='match'>a<\/span><\/li>\n  <li><span class='match'>a<\/span>ble<\/li>\n  <li><span class='match'>a<\/span>bout<\/li>\n  <li><span class='match'>a<\/span>bsolute<\/li>\n  <li><span class='match'>a<\/span>ccept<\/li>\n  <li><span class='match'>a<\/span>ccount<\/li>\n  <li><span class='match'>a<\/span>chieve<\/li>\n  <li><span class='match'>a<\/span>cross<\/li>\n  <li><span class='match'>a<\/span>ct<\/li>\n  <li><span class='match'>a<\/span>ctive<\/li>\n  <li><span class='match'>a<\/span>ctual<\/li>\n  <li><span class='match'>a<\/span>dd<\/li>\n  <li><span class='match'>a<\/span>ddress<\/li>\n  <li><span class='match'>a<\/span>dmit<\/li>\n  <li><span class='match'>a<\/span>dvertise<\/li>\n  <li><span class='match'>a<\/span>ffect<\/li>\n  <li><span class='match'>a<\/span>fford<\/li>\n  <li><span class='match'>a<\/span>fter<\/li>\n  <li><span class='match'>a<\/span>fternoon<\/li>\n  <li><span class='match'>a<\/span>gain<\/li>\n  <li><span class='match'>a<\/span>gainst<\/li>\n  <li><span class='match'>a<\/span>ge<\/li>\n  <li><span class='match'>a<\/span>gent<\/li>\n  <li><span class='match'>a<\/span>go<\/li>\n  <li><span class='match'>a<\/span>gree<\/li>\n  <li><span class='match'>a<\/span>ir<\/li>\n  <li><span class='match'>a<\/span>ll<\/li>\n  <li><span class='match'>a<\/span>llow<\/li>\n  <li><span class='match'>a<\/span>lmost<\/li>\n  <li><span class='match'>a<\/span>long<\/li>\n  <li><span class='match'>a<\/span>lready<\/li>\n  <li><span class='match'>a<\/span>lright<\/li>\n  <li><span class='match'>a<\/span>lso<\/li>\n  <li><span class='match'>a<\/span>lthough<\/li>\n  <li><span class='match'>a<\/span>lways<\/li>\n  <li><span class='match'>a<\/span>merica<\/li>\n  <li><span class='match'>a<\/span>mount<\/li>\n  <li><span class='match'>a<\/span>nd<\/li>\n  <li><span class='match'>a<\/span>nother<\/li>\n  <li><span class='match'>a<\/span>nswer<\/li>\n  <li><span class='match'>a<\/span>ny<\/li>\n  <li><span class='match'>a<\/span>part<\/li>\n  <li><span class='match'>a<\/span>pparent<\/li>\n  <li><span class='match'>a<\/span>ppear<\/li>\n  <li><span class='match'>a<\/span>pply<\/li>\n  <li><span class='match'>a<\/span>ppoint<\/li>\n  <li><span class='match'>a<\/span>pproach<\/li>\n  <li><span class='match'>a<\/span>ppropriate<\/li>\n  <li><span class='match'>a<\/span>rea<\/li>\n  <li><span class='match'>a<\/span>rgue<\/li>\n  <li><span class='match'>a<\/span>rm<\/li>\n  <li><span class='match'>a<\/span>round<\/li>\n  <li><span class='match'>a<\/span>rrange<\/li>\n  <li><span class='match'>a<\/span>rt<\/li>\n  <li><span class='match'>a<\/span>s<\/li>\n  <li><span class='match'>a<\/span>sk<\/li>\n  <li><span class='match'>a<\/span>ssociate<\/li>\n  <li><span class='match'>a<\/span>ssume<\/li>\n  <li><span class='match'>a<\/span>t<\/li>\n  <li><span class='match'>a<\/span>ttend<\/li>\n  <li><span class='match'>a<\/span>uthority<\/li>\n  <li><span class='match'>a<\/span>vailable<\/li>\n  <li><span class='match'>a<\/span>ware<\/li>\n  <li><span class='match'>a<\/span>way<\/li>\n  <li><span class='match'>a<\/span>wful<\/li>\n  <li><span class='match'>e<\/span>ach<\/li>\n  <li><span class='match'>e<\/span>arly<\/li>\n  <li><span class='match'>e<\/span>ast<\/li>\n  <li><span class='match'>e<\/span>asy<\/li>\n  <li><span class='match'>e<\/span>at<\/li>\n  <li><span class='match'>e<\/span>conomy<\/li>\n  <li><span class='match'>e<\/span>ducate<\/li>\n  <li><span class='match'>e<\/span>ffect<\/li>\n  <li><span class='match'>e<\/span>gg<\/li>\n  <li><span class='match'>e<\/span>ight<\/li>\n  <li><span class='match'>e<\/span>ither<\/li>\n  <li><span class='match'>e<\/span>lect<\/li>\n  <li><span class='match'>e<\/span>lectric<\/li>\n  <li><span class='match'>e<\/span>leven<\/li>\n  <li><span class='match'>e<\/span>lse<\/li>\n  <li><span class='match'>e<\/span>mploy<\/li>\n  <li><span class='match'>e<\/span>ncourage<\/li>\n  <li><span class='match'>e<\/span>nd<\/li>\n  <li><span class='match'>e<\/span>ngine<\/li>\n  <li><span class='match'>e<\/span>nglish<\/li>\n  <li><span class='match'>e<\/span>njoy<\/li>\n  <li><span class='match'>e<\/span>nough<\/li>\n  <li><span class='match'>e<\/span>nter<\/li>\n  <li><span class='match'>e<\/span>nvironment<\/li>\n  <li><span class='match'>e<\/span>qual<\/li>\n  <li><span class='match'>e<\/span>special<\/li>\n  <li><span class='match'>e<\/span>urope<\/li>\n  <li><span class='match'>e<\/span>ven<\/li>\n  <li><span class='match'>e<\/span>vening<\/li>\n  <li><span class='match'>e<\/span>ver<\/li>\n  <li><span class='match'>e<\/span>very<\/li>\n  <li><span class='match'>e<\/span>vidence<\/li>\n  <li><span class='match'>e<\/span>xact<\/li>\n  <li><span class='match'>e<\/span>xample<\/li>\n  <li><span class='match'>e<\/span>xcept<\/li>\n  <li><span class='match'>e<\/span>xcuse<\/li>\n  <li><span class='match'>e<\/span>xercise<\/li>\n  <li><span class='match'>e<\/span>xist<\/li>\n  <li><span class='match'>e<\/span>xpect<\/li>\n  <li><span class='match'>e<\/span>xpense<\/li>\n  <li><span class='match'>e<\/span>xperience<\/li>\n  <li><span class='match'>e<\/span>xplain<\/li>\n  <li><span class='match'>e<\/span>xpress<\/li>\n  <li><span class='match'>e<\/span>xtra<\/li>\n  <li><span class='match'>e<\/span>ye<\/li>\n  <li><span class='match'>i<\/span>dea<\/li>\n  <li><span class='match'>i<\/span>dentify<\/li>\n  <li><span class='match'>i<\/span>f<\/li>\n  <li><span class='match'>i<\/span>magine<\/li>\n  <li><span class='match'>i<\/span>mportant<\/li>\n  <li><span class='match'>i<\/span>mprove<\/li>\n  <li><span class='match'>i<\/span>n<\/li>\n  <li><span class='match'>i<\/span>nclude<\/li>\n  <li><span class='match'>i<\/span>ncome<\/li>\n  <li><span class='match'>i<\/span>ncrease<\/li>\n  <li><span class='match'>i<\/span>ndeed<\/li>\n  <li><span class='match'>i<\/span>ndividual<\/li>\n  <li><span class='match'>i<\/span>ndustry<\/li>\n  <li><span class='match'>i<\/span>nform<\/li>\n  <li><span class='match'>i<\/span>nside<\/li>\n  <li><span class='match'>i<\/span>nstead<\/li>\n  <li><span class='match'>i<\/span>nsure<\/li>\n  <li><span class='match'>i<\/span>nterest<\/li>\n  <li><span class='match'>i<\/span>nto<\/li>\n  <li><span class='match'>i<\/span>ntroduce<\/li>\n  <li><span class='match'>i<\/span>nvest<\/li>\n  <li><span class='match'>i<\/span>nvolve<\/li>\n  <li><span class='match'>i<\/span>ssue<\/li>\n  <li><span class='match'>i<\/span>t<\/li>\n  <li><span class='match'>i<\/span>tem<\/li>\n  <li><span class='match'>o<\/span>bvious<\/li>\n  <li><span class='match'>o<\/span>ccasion<\/li>\n  <li><span class='match'>o<\/span>dd<\/li>\n  <li><span class='match'>o<\/span>f<\/li>\n  <li><span class='match'>o<\/span>ff<\/li>\n  <li><span class='match'>o<\/span>ffer<\/li>\n  <li><span class='match'>o<\/span>ffice<\/li>\n  <li><span class='match'>o<\/span>ften<\/li>\n  <li><span class='match'>o<\/span>kay<\/li>\n  <li><span class='match'>o<\/span>ld<\/li>\n  <li><span class='match'>o<\/span>n<\/li>\n  <li><span class='match'>o<\/span>nce<\/li>\n  <li><span class='match'>o<\/span>ne<\/li>\n  <li><span class='match'>o<\/span>nly<\/li>\n  <li><span class='match'>o<\/span>pen<\/li>\n  <li><span class='match'>o<\/span>perate<\/li>\n  <li><span class='match'>o<\/span>pportunity<\/li>\n  <li><span class='match'>o<\/span>ppose<\/li>\n  <li><span class='match'>o<\/span>r<\/li>\n  <li><span class='match'>o<\/span>rder<\/li>\n  <li><span class='match'>o<\/span>rganize<\/li>\n  <li><span class='match'>o<\/span>riginal<\/li>\n  <li><span class='match'>o<\/span>ther<\/li>\n  <li><span class='match'>o<\/span>therwise<\/li>\n  <li><span class='match'>o<\/span>ught<\/li>\n  <li><span class='match'>o<\/span>ut<\/li>\n  <li><span class='match'>o<\/span>ver<\/li>\n  <li><span class='match'>o<\/span>wn<\/li>\n  <li><span class='match'>u<\/span>nder<\/li>\n  <li><span class='match'>u<\/span>nderstand<\/li>\n  <li><span class='match'>u<\/span>nion<\/li>\n  <li><span class='match'>u<\/span>nit<\/li>\n  <li><span class='match'>u<\/span>nite<\/li>\n  <li><span class='match'>u<\/span>niversity<\/li>\n  <li><span class='match'>u<\/span>nless<\/li>\n  <li><span class='match'>u<\/span>ntil<\/li>\n  <li><span class='match'>u<\/span>p<\/li>\n  <li><span class='match'>u<\/span>pon<\/li>\n  <li><span class='match'>u<\/span>se<\/li>\n  <li><span class='match'>u<\/span>sual<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

1.  Words that contain only consonants

    
    ```r
    str_view(stringr::words, "^[^aeiou]+$", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-8799424a7aeb6636ab0f" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-8799424a7aeb6636ab0f">{"x":{"html":"<ul>\n  <li><span class='match'>by<\/span><\/li>\n  <li><span class='match'>dry<\/span><\/li>\n  <li><span class='match'>fly<\/span><\/li>\n  <li><span class='match'>mrs<\/span><\/li>\n  <li><span class='match'>try<\/span><\/li>\n  <li><span class='match'>why<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

    This seems to require using the `+` pattern introduced later, unless one wants to be very verbose and specify words of certain lengths.

1.  Words that end with "-ed" but not ending in "-eed". This handles the special case of "-ed", as well as words with a length great than two.

    
    ```r
    str_view(stringr::words, "^ed$|[^e]ed$", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-ff2684d8b73d8cd50778" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-ff2684d8b73d8cd50778">{"x":{"html":"<ul>\n  <li><span class='match'>bed<\/span><\/li>\n  <li>hund<span class='match'>red<\/span><\/li>\n  <li><span class='match'>red<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

1.  Words ending in `ing` or `ise`:

    
    ```r
    str_view(stringr::words, "i(ng|se)$", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-ced067376bab81a983d5" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-ced067376bab81a983d5">{"x":{"html":"<ul>\n  <li>advert<span class='match'>ise<\/span><\/li>\n  <li>br<span class='match'>ing<\/span><\/li>\n  <li>dur<span class='match'>ing<\/span><\/li>\n  <li>even<span class='match'>ing<\/span><\/li>\n  <li>exerc<span class='match'>ise<\/span><\/li>\n  <li>k<span class='match'>ing<\/span><\/li>\n  <li>mean<span class='match'>ing<\/span><\/li>\n  <li>morn<span class='match'>ing<\/span><\/li>\n  <li>otherw<span class='match'>ise<\/span><\/li>\n  <li>pract<span class='match'>ise<\/span><\/li>\n  <li>ra<span class='match'>ise<\/span><\/li>\n  <li>real<span class='match'>ise<\/span><\/li>\n  <li>r<span class='match'>ing<\/span><\/li>\n  <li>r<span class='match'>ise<\/span><\/li>\n  <li>s<span class='match'>ing<\/span><\/li>\n  <li>surpr<span class='match'>ise<\/span><\/li>\n  <li>th<span class='match'>ing<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

</div>

#### Exercise 14.3.3.2 {.unnumbered .exercise data-number="14.3.3.2"}

<div class="question">

Empirically verify the rule “i” before e except after “c”.

</div>

<div class="answer">

Using only what has been introduced thus far:


```r
str_view(stringr::words, "(cei|[^c]ie)", match = TRUE)
```

<!--html_preserve--><div id="htmlwidget-b5df02e3ff805bc695a2" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-b5df02e3ff805bc695a2">{"x":{"html":"<ul>\n  <li>ac<span class='match'>hie<\/span>ve<\/li>\n  <li>be<span class='match'>lie<\/span>ve<\/li>\n  <li>b<span class='match'>rie<\/span>f<\/li>\n  <li>c<span class='match'>lie<\/span>nt<\/li>\n  <li><span class='match'>die<\/span><\/li>\n  <li>expe<span class='match'>rie<\/span>nce<\/li>\n  <li><span class='match'>fie<\/span>ld<\/li>\n  <li>f<span class='match'>rie<\/span>nd<\/li>\n  <li><span class='match'>lie<\/span><\/li>\n  <li><span class='match'>pie<\/span>ce<\/li>\n  <li>q<span class='match'>uie<\/span>t<\/li>\n  <li>re<span class='match'>cei<\/span>ve<\/li>\n  <li><span class='match'>tie<\/span><\/li>\n  <li><span class='match'>vie<\/span>w<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


```r
str_view(stringr::words, "(cie|[^c]ei)", match = TRUE)
```

<!--html_preserve--><div id="htmlwidget-db9140ebde3f67c51e31" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-db9140ebde3f67c51e31">{"x":{"html":"<ul>\n  <li>s<span class='match'>cie<\/span>nce<\/li>\n  <li>so<span class='match'>cie<\/span>ty<\/li>\n  <li><span class='match'>wei<\/span>gh<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Using `str_detect()` count the number of words that follow these rules:

```r
sum(str_detect(stringr::words, "(cei|[^c]ie)"))
#> [1] 14
sum(str_detect(stringr::words, "(cie|[^c]ei)"))
#> [1] 3
```

</div>

#### Exercise 14.3.3.3 {.unnumbered .exercise data-number="14.3.3.3"}

<div class="question">
Is “q” always followed by a “u”?
</div>

<div class="answer">

In the `stringr::words` dataset, yes. In the full English language, no.

```r
str_view(stringr::words, "q[^u]", match = TRUE)
```

<!--html_preserve--><div id="htmlwidget-2aa9dbed8d93af3e0be8" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-2aa9dbed8d93af3e0be8">{"x":{"html":"<ul>\n  <li><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

</div>

#### Exercise 14.3.3.4 {.unnumbered .exercise data-number="14.3.3.4"}

<div class="question">
Write a regular expression that matches a word if it’s probably written in British English, not American English.
</div>

<div class="answer">

In the general case, this is hard, and could require a dictionary.
But, there are a few heuristics to consider that would account for some common cases: British English tends to use the following:

-   "ou" instead of "o"
-   use of "ae" and "oe" instead of "a" and "o"
-   ends in `ise` instead of `ize`
-   ends in `yse`

The regex `ou|ise$|ae|oe|yse$` would match these.

There are other [spelling differences between American and British English](https://en.wikipedia.org/wiki/American_and_British_English_spelling_differences) but they are not patterns amenable to regular expressions.
It would require a dictionary with differences in spellings for different words.

</div>

#### Exercise 14.3.3.5 {.unnumbered .exercise data-number="14.3.3.5"}

<div class="question">
Create a regular expression that will match telephone numbers as commonly written in your country.
</div>

<div class="answer">

The answer to this will vary by country.

For the United States, phone numbers have a format like `123-456-7890`.

```r
x <- c("123-456-7890", "1235-2351")
str_view(x, "\\d\\d\\d-\\d\\d\\d-\\d\\d\\d\\d")
```

<!--html_preserve--><div id="htmlwidget-12ff9c2ce809973ce8d1" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-12ff9c2ce809973ce8d1">{"x":{"html":"<ul>\n  <li><span class='match'>123-456-7890<\/span><\/li>\n  <li>1235-2351<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
or

```r
str_view(x, "[0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9][0-9][0-9]")
```

<!--html_preserve--><div id="htmlwidget-b338ba377455914076ea" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-b338ba377455914076ea">{"x":{"html":"<ul>\n  <li>123-456-7890<\/li>\n  <li>1235-2351<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

This regular expression can be simplified with the `{m,n}` regular expression modifier introduced in the next section,

```r
str_view(x, "\\d{3}-\\d{3}-\\d{4}")
```

<!--html_preserve--><div id="htmlwidget-f9d1e794c5feb6374ab8" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-f9d1e794c5feb6374ab8">{"x":{"html":"<ul>\n  <li><span class='match'>123-456-7890<\/span><\/li>\n  <li>1235-2351<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Note that this pattern doesn't account for phone numbers that are invalid
because of unassigned area code, or special numbers like 911, or extensions.
See the Wikipedia page for the [North American Numbering
Plan](https://en.wikipedia.org/wiki/North_American_Numbering_Plan) for more
information on the complexities of US phone numbers, and [this Stack Overflow
question](http://stackoverflow.com/questions/123559/a-comprehensive-regex-for-phone-number-validation)
for a discussion of using a regex for phone number validation.

</div>

### Repetition {#repetition .r4ds-section}

#### Exercise 14.3.4.1 {.unnumbered .exercise data-number="14.3.4.1"}

<div class="question">
Describe the equivalents of `?`, `+`, `*` in `{m,n}` form.
</div>

<div class="answer">

| Pattern | `{m,n}` | Meaning           |
|---------|---------|-------------------|
| `?`     | `{0,1}` | Match at most 1   |
| `+`     | `{1,}`  | Match 1 or more   |
| `*`     | `{0,}`  | Match 0 or more   |

For example, let's repeat the examples in the chapter, replacing `?` with `{0,1}`, 
`+` with `{1,}`, and `*` with `{*,}`.

```r
x <- "1888 is the longest year in Roman numerals: MDCCCLXXXVIII"
```

```r
str_view(x, "CC?")
```

<!--html_preserve--><div id="htmlwidget-dd3d01f170c0aa6859bc" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-dd3d01f170c0aa6859bc">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MD<span class='match'>CC<\/span>CLXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view(x, "CC{0,1}")
```

<!--html_preserve--><div id="htmlwidget-aa15db13da1b7c3faf29" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-aa15db13da1b7c3faf29">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MD<span class='match'>CC<\/span>CLXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


```r
str_view(x, "CC+")
```

<!--html_preserve--><div id="htmlwidget-f3525ce3d319e7c562ff" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-f3525ce3d319e7c562ff">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MD<span class='match'>CCC<\/span>LXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view(x, "CC{1,}")
```

<!--html_preserve--><div id="htmlwidget-59f2370825da36350af1" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-59f2370825da36350af1">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MD<span class='match'>CCC<\/span>LXXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


```r
str_view_all(x, "C[LX]+")
```

<!--html_preserve--><div id="htmlwidget-3ec749e04bfb96c2d6c3" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-3ec749e04bfb96c2d6c3">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MDCC<span class='match'>CLXXX<\/span>VIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view_all(x, "C[LX]{0,1}")
```

<!--html_preserve--><div id="htmlwidget-6a2314a79aa854fab7df" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-6a2314a79aa854fab7df">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MD<span class='match'>C<\/span><span class='match'>C<\/span><span class='match'>CL<\/span>XXXVIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

The chapter does not contain an example of `*`.
This pattern looks for a "C" optionally followed by
any number of "L" or "X" characters.

```r
str_view_all(x, "C[LX]*")
```

<!--html_preserve--><div id="htmlwidget-fb37aa630b9d9968db84" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-fb37aa630b9d9968db84">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MD<span class='match'>C<\/span><span class='match'>C<\/span><span class='match'>CLXXX<\/span>VIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

```r
str_view_all(x, "C[LX]{0,}")
```

<!--html_preserve--><div id="htmlwidget-fa04ac5febadaac18a3d" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-fa04ac5febadaac18a3d">{"x":{"html":"<ul>\n  <li>1888 is the longest year in Roman numerals: MD<span class='match'>C<\/span><span class='match'>C<\/span><span class='match'>CLXXX<\/span>VIII<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

</div>

#### Exercise 14.3.4.2 {.unnumbered .exercise data-number="14.3.4.2"}

<div class="question">
Describe in words what these regular expressions match: (read carefully to see if I’m using a regular expression or a string that defines a regular expression.)

1.  `^.*$`
1.  `"\\{.+\\}"`
1.  `\d{4}-\d{2}-\d{2}`
1.  `"\\\\{4}"`

</div>

<div class="answer">

The answer to each part follows.

1.  `^.*$` will match any string. For example: `^.*$`: `c("dog", "$1.23", "lorem ipsum")`.

1.  `"\\{.+\\}"` will match any string with curly braces surrounding at least one character.
    For example: `"\\{.+\\}"`: `c("{a}", "{abc}")`.

1.  `\d{4}-\d{2}-\d{2}` will match four digits followed by a hyphen, followed by
     two digits followed by a hyphen, followed by another two digits.
     This is a regular expression that can match dates formatted like "YYYY-MM-DD" ("%Y-%m-%d").
     For example: `\d{4}-\d{2}-\d{2}`: `2018-01-11`

1.  `"\\\\{4}"` is `\\{4}`, which will match four backslashes.
    For example: `"\\\\{4}"`: `"\\\\\\\\"`.

</div>

#### Exercise 14.3.4.3 {.unnumbered .exercise data-number="14.3.4.3"}

<div class="question">
Create regular expressions to find all words that:

1.  Start with three consonants.
1.  Have three or more vowels in a row.
1.  Have two or more vowel-consonant pairs in a row.

</div>

<div class="answer">

The answer to each part follows.

1.  This regex finds all words starting with three consonants.

    
    ```r
    str_view(words, "^[^aeiou]{3}", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-826abaa365f54c0c9337" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-826abaa365f54c0c9337">{"x":{"html":"<ul>\n  <li><span class='match'>Chr<\/span>ist<\/li>\n  <li><span class='match'>Chr<\/span>istmas<\/li>\n  <li><span class='match'>dry<\/span><\/li>\n  <li><span class='match'>fly<\/span><\/li>\n  <li><span class='match'>mrs<\/span><\/li>\n  <li><span class='match'>sch<\/span>eme<\/li>\n  <li><span class='match'>sch<\/span>ool<\/li>\n  <li><span class='match'>str<\/span>aight<\/li>\n  <li><span class='match'>str<\/span>ategy<\/li>\n  <li><span class='match'>str<\/span>eet<\/li>\n  <li><span class='match'>str<\/span>ike<\/li>\n  <li><span class='match'>str<\/span>ong<\/li>\n  <li><span class='match'>str<\/span>ucture<\/li>\n  <li><span class='match'>sys<\/span>tem<\/li>\n  <li><span class='match'>thr<\/span>ee<\/li>\n  <li><span class='match'>thr<\/span>ough<\/li>\n  <li><span class='match'>thr<\/span>ow<\/li>\n  <li><span class='match'>try<\/span><\/li>\n  <li><span class='match'>typ<\/span>e<\/li>\n  <li><span class='match'>why<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

1.  This regex finds three or more vowels in a row:

    
    ```r
    str_view(words, "[aeiou]{3,}", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-20f0cdc2888b18632cb0" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-20f0cdc2888b18632cb0">{"x":{"html":"<ul>\n  <li>b<span class='match'>eau<\/span>ty<\/li>\n  <li>obv<span class='match'>iou<\/span>s<\/li>\n  <li>prev<span class='match'>iou<\/span>s<\/li>\n  <li>q<span class='match'>uie<\/span>t<\/li>\n  <li>ser<span class='match'>iou<\/span>s<\/li>\n  <li>var<span class='match'>iou<\/span>s<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

1.  This regex finds two or more vowel-consonant pairs in a row.

    
    ```r
    str_view(words, "([aeiou][^aeiou]){2,}", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-acf232f763a6d01286c3" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-acf232f763a6d01286c3">{"x":{"html":"<ul>\n  <li>abs<span class='match'>olut<\/span>e<\/li>\n  <li><span class='match'>agen<\/span>t<\/li>\n  <li><span class='match'>alon<\/span>g<\/li>\n  <li><span class='match'>americ<\/span>a<\/li>\n  <li><span class='match'>anot<\/span>her<\/li>\n  <li><span class='match'>apar<\/span>t<\/li>\n  <li>app<span class='match'>aren<\/span>t<\/li>\n  <li>auth<span class='match'>orit<\/span>y<\/li>\n  <li>ava<span class='match'>ilab<\/span>le<\/li>\n  <li><span class='match'>awar<\/span>e<\/li>\n  <li><span class='match'>away<\/span><\/li>\n  <li>b<span class='match'>alan<\/span>ce<\/li>\n  <li>b<span class='match'>asis<\/span><\/li>\n  <li>b<span class='match'>ecom<\/span>e<\/li>\n  <li>b<span class='match'>efor<\/span>e<\/li>\n  <li>b<span class='match'>egin<\/span><\/li>\n  <li>b<span class='match'>ehin<\/span>d<\/li>\n  <li>b<span class='match'>enefit<\/span><\/li>\n  <li>b<span class='match'>usines<\/span>s<\/li>\n  <li>ch<span class='match'>arac<\/span>ter<\/li>\n  <li>cl<span class='match'>oses<\/span><\/li>\n  <li>comm<span class='match'>unit<\/span>y<\/li>\n  <li>cons<span class='match'>ider<\/span><\/li>\n  <li>c<span class='match'>over<\/span><\/li>\n  <li>d<span class='match'>ebat<\/span>e<\/li>\n  <li>d<span class='match'>ecid<\/span>e<\/li>\n  <li>d<span class='match'>ecis<\/span>ion<\/li>\n  <li>d<span class='match'>efinit<\/span>e<\/li>\n  <li>d<span class='match'>epar<\/span>tment<\/li>\n  <li>d<span class='match'>epen<\/span>d<\/li>\n  <li>d<span class='match'>esig<\/span>n<\/li>\n  <li>d<span class='match'>evelop<\/span><\/li>\n  <li>diff<span class='match'>eren<\/span>ce<\/li>\n  <li>diff<span class='match'>icul<\/span>t<\/li>\n  <li>d<span class='match'>irec<\/span>t<\/li>\n  <li>d<span class='match'>ivid<\/span>e<\/li>\n  <li>d<span class='match'>ocumen<\/span>t<\/li>\n  <li>d<span class='match'>urin<\/span>g<\/li>\n  <li><span class='match'>econom<\/span>y<\/li>\n  <li><span class='match'>educat<\/span>e<\/li>\n  <li><span class='match'>elec<\/span>t<\/li>\n  <li><span class='match'>elec<\/span>tric<\/li>\n  <li><span class='match'>eleven<\/span><\/li>\n  <li>enco<span class='match'>urag<\/span>e<\/li>\n  <li>env<span class='match'>iron<\/span>ment<\/li>\n  <li>e<span class='match'>urop<\/span>e<\/li>\n  <li><span class='match'>even<\/span><\/li>\n  <li><span class='match'>evenin<\/span>g<\/li>\n  <li><span class='match'>ever<\/span><\/li>\n  <li><span class='match'>ever<\/span>y<\/li>\n  <li><span class='match'>eviden<\/span>ce<\/li>\n  <li><span class='match'>exac<\/span>t<\/li>\n  <li><span class='match'>exam<\/span>ple<\/li>\n  <li><span class='match'>exer<\/span>cise<\/li>\n  <li><span class='match'>exis<\/span>t<\/li>\n  <li>f<span class='match'>amil<\/span>y<\/li>\n  <li>f<span class='match'>igur<\/span>e<\/li>\n  <li>f<span class='match'>inal<\/span><\/li>\n  <li>f<span class='match'>inan<\/span>ce<\/li>\n  <li>f<span class='match'>inis<\/span>h<\/li>\n  <li>fr<span class='match'>iday<\/span><\/li>\n  <li>f<span class='match'>utur<\/span>e<\/li>\n  <li>g<span class='match'>eneral<\/span><\/li>\n  <li>g<span class='match'>over<\/span>n<\/li>\n  <li>h<span class='match'>oliday<\/span><\/li>\n  <li>h<span class='match'>ones<\/span>t<\/li>\n  <li>hosp<span class='match'>ital<\/span><\/li>\n  <li>h<span class='match'>owever<\/span><\/li>\n  <li><span class='match'>iden<\/span>tify<\/li>\n  <li><span class='match'>imagin<\/span>e<\/li>\n  <li>ind<span class='match'>ivid<\/span>ual<\/li>\n  <li>int<span class='match'>eres<\/span>t<\/li>\n  <li>intr<span class='match'>oduc<\/span>e<\/li>\n  <li><span class='match'>item<\/span><\/li>\n  <li>j<span class='match'>esus<\/span><\/li>\n  <li>l<span class='match'>evel<\/span><\/li>\n  <li>l<span class='match'>ikel<\/span>y<\/li>\n  <li>l<span class='match'>imit<\/span><\/li>\n  <li>l<span class='match'>ocal<\/span><\/li>\n  <li>m<span class='match'>ajor<\/span><\/li>\n  <li>m<span class='match'>anag<\/span>e<\/li>\n  <li>me<span class='match'>anin<\/span>g<\/li>\n  <li>me<span class='match'>asur<\/span>e<\/li>\n  <li>m<span class='match'>inis<\/span>ter<\/li>\n  <li>m<span class='match'>inus<\/span><\/li>\n  <li>m<span class='match'>inut<\/span>e<\/li>\n  <li>m<span class='match'>omen<\/span>t<\/li>\n  <li>m<span class='match'>oney<\/span><\/li>\n  <li>m<span class='match'>usic<\/span><\/li>\n  <li>n<span class='match'>atur<\/span>e<\/li>\n  <li>n<span class='match'>eces<\/span>sary<\/li>\n  <li>n<span class='match'>ever<\/span><\/li>\n  <li>n<span class='match'>otic<\/span>e<\/li>\n  <li><span class='match'>okay<\/span><\/li>\n  <li><span class='match'>open<\/span><\/li>\n  <li><span class='match'>operat<\/span>e<\/li>\n  <li>opport<span class='match'>unit<\/span>y<\/li>\n  <li>org<span class='match'>aniz<\/span>e<\/li>\n  <li><span class='match'>original<\/span><\/li>\n  <li><span class='match'>over<\/span><\/li>\n  <li>p<span class='match'>aper<\/span><\/li>\n  <li>p<span class='match'>arag<\/span>raph<\/li>\n  <li>p<span class='match'>aren<\/span>t<\/li>\n  <li>part<span class='match'>icular<\/span><\/li>\n  <li>ph<span class='match'>otog<\/span>raph<\/li>\n  <li>p<span class='match'>olic<\/span>e<\/li>\n  <li>p<span class='match'>olic<\/span>y<\/li>\n  <li>p<span class='match'>olitic<\/span><\/li>\n  <li>p<span class='match'>osit<\/span>ion<\/li>\n  <li>p<span class='match'>ositiv<\/span>e<\/li>\n  <li>p<span class='match'>ower<\/span><\/li>\n  <li>pr<span class='match'>epar<\/span>e<\/li>\n  <li>pr<span class='match'>esen<\/span>t<\/li>\n  <li>pr<span class='match'>esum<\/span>e<\/li>\n  <li>pr<span class='match'>ivat<\/span>e<\/li>\n  <li>pr<span class='match'>obab<\/span>le<\/li>\n  <li>pr<span class='match'>oces<\/span>s<\/li>\n  <li>pr<span class='match'>oduc<\/span>e<\/li>\n  <li>pr<span class='match'>oduc<\/span>t<\/li>\n  <li>pr<span class='match'>ojec<\/span>t<\/li>\n  <li>pr<span class='match'>oper<\/span><\/li>\n  <li>pr<span class='match'>opos<\/span>e<\/li>\n  <li>pr<span class='match'>otec<\/span>t<\/li>\n  <li>pr<span class='match'>ovid<\/span>e<\/li>\n  <li>qu<span class='match'>alit<\/span>y<\/li>\n  <li>re<span class='match'>alis<\/span>e<\/li>\n  <li>re<span class='match'>ason<\/span><\/li>\n  <li>r<span class='match'>ecen<\/span>t<\/li>\n  <li>r<span class='match'>ecog<\/span>nize<\/li>\n  <li>r<span class='match'>ecom<\/span>mend<\/li>\n  <li>r<span class='match'>ecor<\/span>d<\/li>\n  <li>r<span class='match'>educ<\/span>e<\/li>\n  <li>r<span class='match'>efer<\/span><\/li>\n  <li>r<span class='match'>egar<\/span>d<\/li>\n  <li>r<span class='match'>elat<\/span>ion<\/li>\n  <li>r<span class='match'>emem<\/span>ber<\/li>\n  <li>r<span class='match'>epor<\/span>t<\/li>\n  <li>repr<span class='match'>esen<\/span>t<\/li>\n  <li>r<span class='match'>esul<\/span>t<\/li>\n  <li>r<span class='match'>etur<\/span>n<\/li>\n  <li>s<span class='match'>atur<\/span>day<\/li>\n  <li>s<span class='match'>econ<\/span>d<\/li>\n  <li>secr<span class='match'>etar<\/span>y<\/li>\n  <li>s<span class='match'>ecur<\/span>e<\/li>\n  <li>s<span class='match'>eparat<\/span>e<\/li>\n  <li>s<span class='match'>even<\/span><\/li>\n  <li>s<span class='match'>imilar<\/span><\/li>\n  <li>sp<span class='match'>ecific<\/span><\/li>\n  <li>str<span class='match'>ateg<\/span>y<\/li>\n  <li>st<span class='match'>uden<\/span>t<\/li>\n  <li>st<span class='match'>upid<\/span><\/li>\n  <li>t<span class='match'>elep<\/span>hone<\/li>\n  <li>t<span class='match'>elevis<\/span>ion<\/li>\n  <li>th<span class='match'>erefor<\/span>e<\/li>\n  <li>tho<span class='match'>usan<\/span>d<\/li>\n  <li>t<span class='match'>oday<\/span><\/li>\n  <li>t<span class='match'>oget<\/span>her<\/li>\n  <li>t<span class='match'>omor<\/span>row<\/li>\n  <li>t<span class='match'>onig<\/span>ht<\/li>\n  <li>t<span class='match'>otal<\/span><\/li>\n  <li>t<span class='match'>owar<\/span>d<\/li>\n  <li>tr<span class='match'>avel<\/span><\/li>\n  <li><span class='match'>unit<\/span><\/li>\n  <li><span class='match'>unit<\/span>e<\/li>\n  <li><span class='match'>univer<\/span>sity<\/li>\n  <li><span class='match'>upon<\/span><\/li>\n  <li>v<span class='match'>isit<\/span><\/li>\n  <li>w<span class='match'>ater<\/span><\/li>\n  <li>w<span class='match'>oman<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

</div>

#### Exercise 14.3.4.4 {.unnumbered .exercise data-number="14.3.4.4"}

<div class="question">

Solve the beginner regexp crosswords at <https://regexcrossword.com/challenges/>

</div>

<div class="answer">

Exercise left to reader. That site validates its solutions, so they aren't repeated here.

</div>

### Grouping and backreferences {#grouping-and-backreferences .r4ds-section}

#### Exercise 14.3.5.1 {.unnumbered .exercise data-number="14.3.5.1"}

<div class="question">
Describe, in words, what these expressions will match:

1.  `(.)\1\1` :
1.  `"(.)(.)\\2\\1"`:
1.  `(..)\1`: 
1.  `"(.).\\1.\\1"`:
1.  `"(.)(.)(.).*\\3\\2\\1"`

</div>

<div class="answer">

The answer to each part follows.

1.  `(.)\1\1`: The same character appearing three times in a row. E.g. `"aaa"`
1.  `"(.)(.)\\2\\1"`: A pair of characters followed by the same pair of characters in reversed order. E.g. `"abba"`.
1.  `(..)\1`: Any two characters repeated. E.g. `"a1a1"`.
1.  `"(.).\\1.\\1"`: A character followed by any character, the original character, any other character, the original character again. E.g. `"abaca"`, `"b8b.b"`.
1.  `"(.)(.)(.).*\\3\\2\\1"` Three characters followed by zero or more characters of any kind followed by the same three characters but in reverse order. E.g. `"abcsgasgddsadgsdgcba"` or `"abccba"` or `"abc1cba"`.

</div>

#### Exercise 14.3.5.2 {.unnumbered .exercise data-number="14.3.5.2"}

<div class="question">
Construct regular expressions to match words that:

1.  Start and end with the same character.
1.  Contain a repeated pair of letters (e.g. ``church'' contains ``ch'' repeated twice.)
1.  Contain one letter repeated in at least three places (e.g. ``eleven'' contains three ``e''s.)

</div>

<div class="answer">

The answer to each part follows.

1.  This regular expression matches words that and end with the same character.

    
    ```r
    str_view(stringr::words, "^(.)((.*\\1$)|\\1?$)", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-6f8d3407f82dc7e2d69a" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-6f8d3407f82dc7e2d69a">{"x":{"html":"<ul>\n  <li><span class='match'>a<\/span><\/li>\n  <li><span class='match'>america<\/span><\/li>\n  <li><span class='match'>area<\/span><\/li>\n  <li><span class='match'>dad<\/span><\/li>\n  <li><span class='match'>dead<\/span><\/li>\n  <li><span class='match'>depend<\/span><\/li>\n  <li><span class='match'>educate<\/span><\/li>\n  <li><span class='match'>else<\/span><\/li>\n  <li><span class='match'>encourage<\/span><\/li>\n  <li><span class='match'>engine<\/span><\/li>\n  <li><span class='match'>europe<\/span><\/li>\n  <li><span class='match'>evidence<\/span><\/li>\n  <li><span class='match'>example<\/span><\/li>\n  <li><span class='match'>excuse<\/span><\/li>\n  <li><span class='match'>exercise<\/span><\/li>\n  <li><span class='match'>expense<\/span><\/li>\n  <li><span class='match'>experience<\/span><\/li>\n  <li><span class='match'>eye<\/span><\/li>\n  <li><span class='match'>health<\/span><\/li>\n  <li><span class='match'>high<\/span><\/li>\n  <li><span class='match'>knock<\/span><\/li>\n  <li><span class='match'>level<\/span><\/li>\n  <li><span class='match'>local<\/span><\/li>\n  <li><span class='match'>nation<\/span><\/li>\n  <li><span class='match'>non<\/span><\/li>\n  <li><span class='match'>rather<\/span><\/li>\n  <li><span class='match'>refer<\/span><\/li>\n  <li><span class='match'>remember<\/span><\/li>\n  <li><span class='match'>serious<\/span><\/li>\n  <li><span class='match'>stairs<\/span><\/li>\n  <li><span class='match'>test<\/span><\/li>\n  <li><span class='match'>tonight<\/span><\/li>\n  <li><span class='match'>transport<\/span><\/li>\n  <li><span class='match'>treat<\/span><\/li>\n  <li><span class='match'>trust<\/span><\/li>\n  <li><span class='match'>window<\/span><\/li>\n  <li><span class='match'>yesterday<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

1.  Either of these patterns checks for any pair of repeated letters, where
    *letters* is defined to the ASCII letters A-Z.
    
    
    ```r
    str_view(words, "([A-Za-z][A-Za-z]).*\\1", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-e8092118b2671020ef37" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-e8092118b2671020ef37">{"x":{"html":"<ul>\n  <li>ap<span class='match'>propr<\/span>iate<\/li>\n  <li><span class='match'>church<\/span><\/li>\n  <li>c<span class='match'>ondition<\/span><\/li>\n  <li><span class='match'>decide<\/span><\/li>\n  <li><span class='match'>environmen<\/span>t<\/li>\n  <li>l<span class='match'>ondon<\/span><\/li>\n  <li>pa<span class='match'>ragra<\/span>ph<\/li>\n  <li>p<span class='match'>articular<\/span><\/li>\n  <li><span class='match'>photograph<\/span><\/li>\n  <li>p<span class='match'>repare<\/span><\/li>\n  <li>p<span class='match'>ressure<\/span><\/li>\n  <li>r<span class='match'>emem<\/span>ber<\/li>\n  <li><span class='match'>repre<\/span>sent<\/li>\n  <li><span class='match'>require<\/span><\/li>\n  <li><span class='match'>sense<\/span><\/li>\n  <li>the<span class='match'>refore<\/span><\/li>\n  <li>u<span class='match'>nderstand<\/span><\/li>\n  <li>w<span class='match'>hethe<\/span>r<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

    
    ```r
    str_view(words, "([[:letter:]]).*\\1", match = TRUE)
    ```
    
    <!--html_preserve--><div id="htmlwidget-aa348cd320424903da69" style="width:960px;height:100%;" class="str_view html-widget"></div>
    <script type="application/json" data-for="htmlwidget-aa348cd320424903da69">{"x":{"html":"<ul>\n  <li>a<span class='match'>cc<\/span>ept<\/li>\n  <li>a<span class='match'>cc<\/span>ount<\/li>\n  <li>achi<span class='match'>eve<\/span><\/li>\n  <li>acro<span class='match'>ss<\/span><\/li>\n  <li><span class='match'>actua<\/span>l<\/li>\n  <li>a<span class='match'>dd<\/span><\/li>\n  <li>a<span class='match'>dd<\/span>ress<\/li>\n  <li>adv<span class='match'>ertise<\/span><\/li>\n  <li>a<span class='match'>ff<\/span>ect<\/li>\n  <li>a<span class='match'>ff<\/span>ord<\/li>\n  <li>after<span class='match'>noon<\/span><\/li>\n  <li><span class='match'>aga<\/span>in<\/li>\n  <li><span class='match'>aga<\/span>inst<\/li>\n  <li>agr<span class='match'>ee<\/span><\/li>\n  <li>a<span class='match'>ll<\/span><\/li>\n  <li>a<span class='match'>ll<\/span>ow<\/li>\n  <li><span class='match'>alrea<\/span>dy<\/li>\n  <li>alt<span class='match'>hough<\/span><\/li>\n  <li><span class='match'>alwa<\/span>ys<\/li>\n  <li><span class='match'>america<\/span><\/li>\n  <li><span class='match'>apa<\/span>rt<\/li>\n  <li><span class='match'>appa<\/span>rent<\/li>\n  <li><span class='match'>appea<\/span>r<\/li>\n  <li>a<span class='match'>pp<\/span>ly<\/li>\n  <li>a<span class='match'>pp<\/span>oint<\/li>\n  <li><span class='match'>approa<\/span>ch<\/li>\n  <li><span class='match'>appropria<\/span>te<\/li>\n  <li><span class='match'>area<\/span><\/li>\n  <li><span class='match'>arra<\/span>nge<\/li>\n  <li><span class='match'>associa<\/span>te<\/li>\n  <li>a<span class='match'>ss<\/span>ume<\/li>\n  <li>a<span class='match'>tt<\/span>end<\/li>\n  <li>au<span class='match'>thorit<\/span>y<\/li>\n  <li><span class='match'>availa<\/span>ble<\/li>\n  <li><span class='match'>awa<\/span>re<\/li>\n  <li><span class='match'>awa<\/span>y<\/li>\n  <li><span class='match'>bab<\/span>y<\/li>\n  <li>b<span class='match'>ala<\/span>nce<\/li>\n  <li>ba<span class='match'>ll<\/span><\/li>\n  <li>ba<span class='match'>sis<\/span><\/li>\n  <li>b<span class='match'>ecause<\/span><\/li>\n  <li>b<span class='match'>ecome<\/span><\/li>\n  <li>b<span class='match'>efore<\/span><\/li>\n  <li>b<span class='match'>elieve<\/span><\/li>\n  <li>b<span class='match'>ene<\/span>fit<\/li>\n  <li>b<span class='match'>etwee<\/span>n<\/li>\n  <li>bi<span class='match'>ll<\/span><\/li>\n  <li>bl<span class='match'>oo<\/span>d<\/li>\n  <li>b<span class='match'>oo<\/span>k<\/li>\n  <li>bo<span class='match'>tt<\/span>le<\/li>\n  <li>b<span class='match'>otto<\/span>m<\/li>\n  <li>br<span class='match'>illi<\/span>ant<\/li>\n  <li>br<span class='match'>itai<\/span>n<\/li>\n  <li>b<span class='match'>rother<\/span><\/li>\n  <li>bu<span class='match'>siness<\/span><\/li>\n  <li>ca<span class='match'>ll<\/span><\/li>\n  <li>ca<span class='match'>rr<\/span>y<\/li>\n  <li><span class='match'>catc<\/span>h<\/li>\n  <li>c<span class='match'>entre<\/span><\/li>\n  <li>ch<span class='match'>airma<\/span>n<\/li>\n  <li><span class='match'>chanc<\/span>e<\/li>\n  <li><span class='match'>charac<\/span>ter<\/li>\n  <li><span class='match'>chec<\/span>k<\/li>\n  <li><span class='match'>choic<\/span>e<\/li>\n  <li>ch<span class='match'>oo<\/span>se<\/li>\n  <li>Chri<span class='match'>stmas<\/span><\/li>\n  <li><span class='match'>churc<\/span>h<\/li>\n  <li>cla<span class='match'>ss<\/span><\/li>\n  <li><span class='match'>cloc<\/span>k<\/li>\n  <li>clo<span class='match'>ses<\/span><\/li>\n  <li>co<span class='match'>ff<\/span>ee<\/li>\n  <li>co<span class='match'>ll<\/span>eague<\/li>\n  <li><span class='match'>collec<\/span>t<\/li>\n  <li>co<span class='match'>ll<\/span>ege<\/li>\n  <li>c<span class='match'>olo<\/span>ur<\/li>\n  <li>co<span class='match'>mm<\/span>ent<\/li>\n  <li>co<span class='match'>mm<\/span>it<\/li>\n  <li>co<span class='match'>mm<\/span>ittee<\/li>\n  <li>c<span class='match'>ommo<\/span>n<\/li>\n  <li>co<span class='match'>mm<\/span>unity<\/li>\n  <li>compl<span class='match'>ete<\/span><\/li>\n  <li><span class='match'>conc<\/span>ern<\/li>\n  <li>c<span class='match'>onditio<\/span>n<\/li>\n  <li><span class='match'>contac<\/span>t<\/li>\n  <li>co<span class='match'>ntin<\/span>ue<\/li>\n  <li><span class='match'>contrac<\/span>t<\/li>\n  <li>c<span class='match'>ontro<\/span>l<\/li>\n  <li>conv<span class='match'>erse<\/span><\/li>\n  <li>c<span class='match'>oo<\/span>k<\/li>\n  <li>co<span class='match'>rner<\/span><\/li>\n  <li><span class='match'>correc<\/span>t<\/li>\n  <li><span class='match'>counc<\/span>il<\/li>\n  <li>cr<span class='match'>eate<\/span><\/li>\n  <li>cro<span class='match'>ss<\/span><\/li>\n  <li>cu<span class='match'>rr<\/span>ent<\/li>\n  <li><span class='match'>dad<\/span><\/li>\n  <li><span class='match'>dead<\/span><\/li>\n  <li>d<span class='match'>ebate<\/span><\/li>\n  <li><span class='match'>decid<\/span>e<\/li>\n  <li>dec<span class='match'>isi<\/span>on<\/li>\n  <li>d<span class='match'>ee<\/span>p<\/li>\n  <li>d<span class='match'>efinite<\/span><\/li>\n  <li>d<span class='match'>egree<\/span><\/li>\n  <li>d<span class='match'>epartme<\/span>nt<\/li>\n  <li><span class='match'>depend<\/span><\/li>\n  <li>d<span class='match'>escribe<\/span><\/li>\n  <li>d<span class='match'>eve<\/span>lop<\/li>\n  <li>di<span class='match'>ff<\/span>erence<\/li>\n  <li>d<span class='match'>iffi<\/span>cult<\/li>\n  <li>di<span class='match'>nn<\/span>er<\/li>\n  <li>di<span class='match'>scuss<\/span><\/li>\n  <li>d<span class='match'>istri<\/span>ct<\/li>\n  <li><span class='match'>divid<\/span>e<\/li>\n  <li>d<span class='match'>octo<\/span>r<\/li>\n  <li>d<span class='match'>oo<\/span>r<\/li>\n  <li>dre<span class='match'>ss<\/span><\/li>\n  <li>ec<span class='match'>ono<\/span>my<\/li>\n  <li><span class='match'>educate<\/span><\/li>\n  <li><span class='match'>effe<\/span>ct<\/li>\n  <li>e<span class='match'>gg<\/span><\/li>\n  <li><span class='match'>eithe<\/span>r<\/li>\n  <li><span class='match'>ele<\/span>ct<\/li>\n  <li><span class='match'>ele<\/span>ctric<\/li>\n  <li><span class='match'>eleve<\/span>n<\/li>\n  <li><span class='match'>else<\/span><\/li>\n  <li><span class='match'>encourage<\/span><\/li>\n  <li><span class='match'>engine<\/span><\/li>\n  <li><span class='match'>ente<\/span>r<\/li>\n  <li><span class='match'>environme<\/span>nt<\/li>\n  <li><span class='match'>espe<\/span>cial<\/li>\n  <li><span class='match'>europe<\/span><\/li>\n  <li><span class='match'>eve<\/span>n<\/li>\n  <li><span class='match'>eve<\/span>ning<\/li>\n  <li><span class='match'>eve<\/span>r<\/li>\n  <li><span class='match'>eve<\/span>ry<\/li>\n  <li><span class='match'>evidence<\/span><\/li>\n  <li><span class='match'>example<\/span><\/li>\n  <li><span class='match'>exce<\/span>pt<\/li>\n  <li><span class='match'>excuse<\/span><\/li>\n  <li><span class='match'>exercise<\/span><\/li>\n  <li><span class='match'>expe<\/span>ct<\/li>\n  <li><span class='match'>expense<\/span><\/li>\n  <li><span class='match'>experience<\/span><\/li>\n  <li><span class='match'>expre<\/span>ss<\/li>\n  <li><span class='match'>eye<\/span><\/li>\n  <li>fa<span class='match'>ll<\/span><\/li>\n  <li>f<span class='match'>ee<\/span>d<\/li>\n  <li>f<span class='match'>ee<\/span>l<\/li>\n  <li>fi<span class='match'>ll<\/span><\/li>\n  <li>fi<span class='match'>nan<\/span>ce<\/li>\n  <li>f<span class='match'>ini<\/span>sh<\/li>\n  <li>fl<span class='match'>oo<\/span>r<\/li>\n  <li>f<span class='match'>ollo<\/span>w<\/li>\n  <li>f<span class='match'>oo<\/span>d<\/li>\n  <li>f<span class='match'>oo<\/span>t<\/li>\n  <li>fo<span class='match'>rwar<\/span>d<\/li>\n  <li>fr<span class='match'>ee<\/span><\/li>\n  <li>fu<span class='match'>ll<\/span><\/li>\n  <li>fu<span class='match'>nction<\/span><\/li>\n  <li>fu<span class='match'>rther<\/span><\/li>\n  <li>f<span class='match'>utu<\/span>re<\/li>\n  <li>g<span class='match'>ene<\/span>ral<\/li>\n  <li>gla<span class='match'>ss<\/span><\/li>\n  <li>g<span class='match'>oo<\/span>d<\/li>\n  <li>g<span class='match'>oo<\/span>dbye<\/li>\n  <li>gr<span class='match'>ee<\/span>n<\/li>\n  <li>gue<span class='match'>ss<\/span><\/li>\n  <li>ha<span class='match'>ll<\/span><\/li>\n  <li>ha<span class='match'>pp<\/span>en<\/li>\n  <li>ha<span class='match'>pp<\/span>y<\/li>\n  <li><span class='match'>health<\/span><\/li>\n  <li>he<span class='match'>ll<\/span><\/li>\n  <li>h<span class='match'>ere<\/span><\/li>\n  <li><span class='match'>high<\/span><\/li>\n  <li>how<span class='match'>eve<\/span>r<\/li>\n  <li>hu<span class='match'>ll<\/span>o<\/li>\n  <li>hun<span class='match'>dred<\/span><\/li>\n  <li><span class='match'>identi<\/span>fy<\/li>\n  <li><span class='match'>imagi<\/span>ne<\/li>\n  <li>impor<span class='match'>tant<\/span><\/li>\n  <li>incr<span class='match'>ease<\/span><\/li>\n  <li>in<span class='match'>deed<\/span><\/li>\n  <li><span class='match'>indivi<\/span>dual<\/li>\n  <li><span class='match'>insi<\/span>de<\/li>\n  <li>in<span class='match'>terest<\/span><\/li>\n  <li>in<span class='match'>volv<\/span>e<\/li>\n  <li>i<span class='match'>ss<\/span>ue<\/li>\n  <li>je<span class='match'>sus<\/span><\/li>\n  <li>k<span class='match'>ee<\/span>p<\/li>\n  <li>ki<span class='match'>ll<\/span><\/li>\n  <li><span class='match'>knock<\/span><\/li>\n  <li>l<span class='match'>angua<\/span>ge<\/li>\n  <li>l<span class='match'>eave<\/span><\/li>\n  <li>le<span class='match'>ss<\/span><\/li>\n  <li>l<span class='match'>ette<\/span>r<\/li>\n  <li><span class='match'>level<\/span><\/li>\n  <li><span class='match'>likel<\/span>y<\/li>\n  <li>l<span class='match'>imi<\/span>t<\/li>\n  <li><span class='match'>littl<\/span>e<\/li>\n  <li><span class='match'>local<\/span><\/li>\n  <li>l<span class='match'>ondo<\/span>n<\/li>\n  <li>l<span class='match'>oo<\/span>k<\/li>\n  <li>m<span class='match'>ana<\/span>ge<\/li>\n  <li>ma<span class='match'>rr<\/span>y<\/li>\n  <li>ma<span class='match'>tt<\/span>er<\/li>\n  <li>mea<span class='match'>nin<\/span>g<\/li>\n  <li>m<span class='match'>easure<\/span><\/li>\n  <li>m<span class='match'>ee<\/span>t<\/li>\n  <li><span class='match'>mem<\/span>ber<\/li>\n  <li>me<span class='match'>ntion<\/span><\/li>\n  <li>mi<span class='match'>dd<\/span>le<\/li>\n  <li>m<span class='match'>illi<\/span>on<\/li>\n  <li>m<span class='match'>ini<\/span>ster<\/li>\n  <li>mi<span class='match'>ss<\/span><\/li>\n  <li><span class='match'>mom<\/span>ent<\/li>\n  <li>mor<span class='match'>nin<\/span>g<\/li>\n  <li>m<span class='match'>otio<\/span>n<\/li>\n  <li><span class='match'>nation<\/span><\/li>\n  <li>n<span class='match'>ece<\/span>ssary<\/li>\n  <li>n<span class='match'>ee<\/span>d<\/li>\n  <li>n<span class='match'>eve<\/span>r<\/li>\n  <li><span class='match'>nin<\/span>e<\/li>\n  <li><span class='match'>non<\/span><\/li>\n  <li><span class='match'>non<\/span>e<\/li>\n  <li><span class='match'>obvio<\/span>us<\/li>\n  <li><span class='match'>occasio<\/span>n<\/li>\n  <li>o<span class='match'>dd<\/span><\/li>\n  <li>o<span class='match'>ff<\/span><\/li>\n  <li>o<span class='match'>ff<\/span>er<\/li>\n  <li>o<span class='match'>ff<\/span>ice<\/li>\n  <li>op<span class='match'>erate<\/span><\/li>\n  <li><span class='match'>oppo<\/span>rtunity<\/li>\n  <li><span class='match'>oppo<\/span>se<\/li>\n  <li>o<span class='match'>rder<\/span><\/li>\n  <li>or<span class='match'>igi<\/span>nal<\/li>\n  <li>oth<span class='match'>erwise<\/span><\/li>\n  <li><span class='match'>pap<\/span>er<\/li>\n  <li><span class='match'>paragrap<\/span>h<\/li>\n  <li>p<span class='match'>articula<\/span>r<\/li>\n  <li>pa<span class='match'>ss<\/span><\/li>\n  <li>p<span class='match'>ence<\/span><\/li>\n  <li>pe<span class='match'>nsion<\/span><\/li>\n  <li><span class='match'>peop<\/span>le<\/li>\n  <li>p<span class='match'>erce<\/span>nt<\/li>\n  <li>p<span class='match'>erfe<\/span>ct<\/li>\n  <li><span class='match'>perhap<\/span>s<\/li>\n  <li><span class='match'>photograp<\/span>h<\/li>\n  <li>pi<span class='match'>ece<\/span><\/li>\n  <li>pl<span class='match'>ease<\/span><\/li>\n  <li>pol<span class='match'>iti<\/span>c<\/li>\n  <li>p<span class='match'>oo<\/span>r<\/li>\n  <li>p<span class='match'>ositio<\/span>n<\/li>\n  <li>pos<span class='match'>iti<\/span>ve<\/li>\n  <li>po<span class='match'>ss<\/span>ible<\/li>\n  <li><span class='match'>prep<\/span>are<\/li>\n  <li>pr<span class='match'>ese<\/span>nt<\/li>\n  <li>pre<span class='match'>ss<\/span><\/li>\n  <li>p<span class='match'>ressur<\/span>e<\/li>\n  <li>pr<span class='match'>esume<\/span><\/li>\n  <li>pre<span class='match'>tt<\/span>y<\/li>\n  <li>pro<span class='match'>bab<\/span>le<\/li>\n  <li>proc<span class='match'>ee<\/span>d<\/li>\n  <li>proce<span class='match'>ss<\/span><\/li>\n  <li>p<span class='match'>rogr<\/span>amme<\/li>\n  <li><span class='match'>prop<\/span>er<\/li>\n  <li><span class='match'>prop<\/span>ose<\/li>\n  <li>pro<span class='match'>tect<\/span><\/li>\n  <li>pu<span class='match'>ll<\/span><\/li>\n  <li><span class='match'>purp<\/span>ose<\/li>\n  <li>qua<span class='match'>rter<\/span><\/li>\n  <li><span class='match'>rather<\/span><\/li>\n  <li>r<span class='match'>ealise<\/span><\/li>\n  <li>rea<span class='match'>ll<\/span>y<\/li>\n  <li>r<span class='match'>eceive<\/span><\/li>\n  <li>r<span class='match'>ece<\/span>nt<\/li>\n  <li>r<span class='match'>ecognize<\/span><\/li>\n  <li>r<span class='match'>ecomme<\/span>nd<\/li>\n  <li><span class='match'>recor<\/span>d<\/li>\n  <li>r<span class='match'>educe<\/span><\/li>\n  <li><span class='match'>refer<\/span><\/li>\n  <li><span class='match'>regar<\/span>d<\/li>\n  <li><span class='match'>remember<\/span><\/li>\n  <li><span class='match'>repor<\/span>t<\/li>\n  <li><span class='match'>repr<\/span>esent<\/li>\n  <li><span class='match'>requir<\/span>e<\/li>\n  <li><span class='match'>resear<\/span>ch<\/li>\n  <li><span class='match'>resour<\/span>ce<\/li>\n  <li>r<span class='match'>espe<\/span>ct<\/li>\n  <li>r<span class='match'>esponsible<\/span><\/li>\n  <li><span class='match'>retur<\/span>n<\/li>\n  <li>ro<span class='match'>ll<\/span><\/li>\n  <li>r<span class='match'>oo<\/span>m<\/li>\n  <li>s<span class='match'>aturda<\/span>y<\/li>\n  <li>sch<span class='match'>eme<\/span><\/li>\n  <li>sch<span class='match'>oo<\/span>l<\/li>\n  <li>s<span class='match'>cienc<\/span>e<\/li>\n  <li>s<span class='match'>ecre<\/span>tary<\/li>\n  <li>s<span class='match'>ecure<\/span><\/li>\n  <li>s<span class='match'>ee<\/span><\/li>\n  <li>s<span class='match'>ee<\/span>m<\/li>\n  <li>se<span class='match'>ll<\/span><\/li>\n  <li><span class='match'>sens<\/span>e<\/li>\n  <li>s<span class='match'>eparate<\/span><\/li>\n  <li><span class='match'>serious<\/span><\/li>\n  <li>s<span class='match'>erve<\/span><\/li>\n  <li>s<span class='match'>ervice<\/span><\/li>\n  <li>s<span class='match'>ettle<\/span><\/li>\n  <li>s<span class='match'>eve<\/span>n<\/li>\n  <li>sha<span class='match'>ll<\/span><\/li>\n  <li>sh<span class='match'>ee<\/span>t<\/li>\n  <li>sh<span class='match'>oo<\/span>t<\/li>\n  <li>s<span class='match'>imi<\/span>lar<\/li>\n  <li><span class='match'>sis<\/span>ter<\/li>\n  <li>si<span class='match'>tuat<\/span>e<\/li>\n  <li>sl<span class='match'>ee<\/span>p<\/li>\n  <li>sma<span class='match'>ll<\/span><\/li>\n  <li>s<span class='match'>oo<\/span>n<\/li>\n  <li>so<span class='match'>rr<\/span>y<\/li>\n  <li>spe<span class='match'>cific<\/span><\/li>\n  <li>sp<span class='match'>ee<\/span>d<\/li>\n  <li>spe<span class='match'>ll<\/span><\/li>\n  <li>sta<span class='match'>ff<\/span><\/li>\n  <li><span class='match'>stairs<\/span><\/li>\n  <li>st<span class='match'>anda<\/span>rd<\/li>\n  <li>s<span class='match'>tart<\/span><\/li>\n  <li>s<span class='match'>tat<\/span>e<\/li>\n  <li>s<span class='match'>tat<\/span>ion<\/li>\n  <li>sti<span class='match'>ll<\/span><\/li>\n  <li>s<span class='match'>traight<\/span><\/li>\n  <li>s<span class='match'>trat<\/span>egy<\/li>\n  <li>s<span class='match'>treet<\/span><\/li>\n  <li>s<span class='match'>truct<\/span>ure<\/li>\n  <li>s<span class='match'>tudent<\/span><\/li>\n  <li>stu<span class='match'>ff<\/span><\/li>\n  <li>su<span class='match'>cc<\/span>eed<\/li>\n  <li>su<span class='match'>dd<\/span>en<\/li>\n  <li><span class='match'>sugges<\/span>t<\/li>\n  <li>su<span class='match'>mm<\/span>er<\/li>\n  <li>su<span class='match'>pp<\/span>ly<\/li>\n  <li>su<span class='match'>pp<\/span>ort<\/li>\n  <li><span class='match'>suppos<\/span>e<\/li>\n  <li><span class='match'>surpris<\/span>e<\/li>\n  <li><span class='match'>sys<\/span>tem<\/li>\n  <li>t<span class='match'>elephone<\/span><\/li>\n  <li>t<span class='match'>ele<\/span>vision<\/li>\n  <li>te<span class='match'>ll<\/span><\/li>\n  <li>t<span class='match'>errible<\/span><\/li>\n  <li><span class='match'>test<\/span><\/li>\n  <li>th<span class='match'>ere<\/span><\/li>\n  <li>th<span class='match'>erefore<\/span><\/li>\n  <li><span class='match'>thirt<\/span>een<\/li>\n  <li><span class='match'>thirt<\/span>y<\/li>\n  <li>t<span class='match'>hough<\/span><\/li>\n  <li>thr<span class='match'>ee<\/span><\/li>\n  <li>t<span class='match'>hrough<\/span><\/li>\n  <li><span class='match'>toget<\/span>her<\/li>\n  <li>t<span class='match'>omorro<\/span>w<\/li>\n  <li><span class='match'>tonight<\/span><\/li>\n  <li>t<span class='match'>oo<\/span><\/li>\n  <li><span class='match'>tot<\/span>al<\/li>\n  <li>tra<span class='match'>ff<\/span>ic<\/li>\n  <li><span class='match'>transport<\/span><\/li>\n  <li><span class='match'>treat<\/span><\/li>\n  <li>tr<span class='match'>ee<\/span><\/li>\n  <li><span class='match'>trust<\/span><\/li>\n  <li>tw<span class='match'>elve<\/span><\/li>\n  <li><span class='match'>twent<\/span>y<\/li>\n  <li>u<span class='match'>nderstan<\/span>d<\/li>\n  <li>u<span class='match'>nion<\/span><\/li>\n  <li>un<span class='match'>iversi<\/span>ty<\/li>\n  <li>unle<span class='match'>ss<\/span><\/li>\n  <li><span class='match'>usu<\/span>al<\/li>\n  <li>vi<span class='match'>ll<\/span>age<\/li>\n  <li>v<span class='match'>isi<\/span>t<\/li>\n  <li>wa<span class='match'>ll<\/span><\/li>\n  <li>w<span class='match'>edne<\/span>sday<\/li>\n  <li>w<span class='match'>ee<\/span><\/li>\n  <li>w<span class='match'>ee<\/span>k<\/li>\n  <li>w<span class='match'>elcome<\/span><\/li>\n  <li>we<span class='match'>ll<\/span><\/li>\n  <li>wh<span class='match'>ere<\/span><\/li>\n  <li>w<span class='match'>heth<\/span>er<\/li>\n  <li>w<span class='match'>hich<\/span><\/li>\n  <li>wi<span class='match'>ll<\/span><\/li>\n  <li><span class='match'>window<\/span><\/li>\n  <li>w<span class='match'>ithi<\/span>n<\/li>\n  <li>wi<span class='match'>thout<\/span><\/li>\n  <li>w<span class='match'>oo<\/span>d<\/li>\n  <li>wo<span class='match'>rr<\/span>y<\/li>\n  <li><span class='match'>yesterday<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

    The `\\1` pattern is called a backreference. It matches whatever the first group
    matched. This allows the pattern to match a repeating pair of letters without having
    to specify exactly what pair letters is being repeated.

    Note that these patterns are case sensitive. Use the
    case insensitive flag if you want to check for repeated pairs
    of letters with different capitalization.

1.  This regex matches words that contain one letter repeated in at least three places.

    
    ```r
    str_subset(str_to_lower(words), "([a-z]).*\\1.*\\1")
    #>  [1] "appropriate" "available"   "believe"     "between"     "business"   
    #>  [6] "degree"      "difference"  "discuss"     "eleven"      "environment"
    #> [11] "evidence"    "exercise"    "expense"     "experience"  "individual" 
    #> [16] "paragraph"   "receive"     "remember"    "represent"   "telephone"  
    #> [21] "therefore"   "tomorrow"
    ```

</div>

## Tools {#tools .r4ds-section}

### Detect matches {#detect-matches .r4ds-section}

#### Exercise 14.4.1.1 {.unnumbered .exercise data-number="14.4.1.1"}

<div class="question">

For each of the following challenges, try solving it by using both a single regular expression, and a combination of multiple `str_detect()` calls.

1.  Find all words that start or end with x.
1.  Find all words that start with a vowel and end with a consonant.
1.  Are there any words that contain at least one of each different vowel?

</div>

<div class="answer">

The answer to each part follows.

1.  Words that start or end with `x`?

    
    ```r
    # one regex
    words[str_detect(words, "^x|x$")]
    #> [1] "box" "sex" "six" "tax"
    # split regex into parts
    start_with_x <- str_detect(words, "^x")
    end_with_x <- str_detect(words, "x$")
    words[start_with_x | end_with_x]
    #> [1] "box" "sex" "six" "tax"
    ```

1.  Words starting with vowel and ending with consonant.

    
    ```r
    str_subset(words, "^[aeiou].*[^aeiou]$") %>% head()
    #> [1] "about"   "accept"  "account" "across"  "act"     "actual"
    start_with_vowel <- str_detect(words, "^[aeiou]")
    end_with_consonant <- str_detect(words, "[^aeiou]$")
    words[start_with_vowel & end_with_consonant] %>% head()
    #> [1] "about"   "accept"  "account" "across"  "act"     "actual"
    ```

1.  There is not a simple regular expression to match words that
    that contain at least one of each vowel. The regular expression
    would need to consider all possible orders in which the vowels
    could occur.

    
    ```r
    pattern <-
      cross(rerun(5, c("a", "e", "i", "o", "u")),
        .filter = function(...) {
          x <- as.character(unlist(list(...)))
          length(x) != length(unique(x))
        }
      ) %>%
      map_chr(~ str_c(unlist(.x), collapse = ".*")) %>%
      str_c(collapse = "|")
    ```

    To check that this pattern works, test it on a pattern that
    should match
    
    ```r
    str_subset("aseiouds", pattern)
    #> [1] "aseiouds"
    ```

    Using multiple `str_detect()` calls, one pattern for each vowel,
    produces a much simpler and readable answer.

    
    ```r
    str_subset(words, pattern)
    #> character(0)
    
    words[str_detect(words, "a") &
      str_detect(words, "e") &
      str_detect(words, "i") &
      str_detect(words, "o") &
      str_detect(words, "u")]
    #> character(0)
    ```

    There appear to be none.

</div>

#### Exercise 14.4.1.2 {.unnumbered .exercise data-number="14.4.1.2"}

<div class="question">
  
What word has the higher number of vowels? What word has the highest proportion of vowels? (Hint: what is the denominator?)

</div>

<div class="answer">
  
The word with the highest number of vowels is
    

```r
vowels <- str_count(words, "[aeiou]")
words[which(vowels == max(vowels))]
#> [1] "appropriate" "associate"   "available"   "colleague"   "encourage"  
#> [6] "experience"  "individual"  "television"
```

The word with the highest proportion of vowels is

```r
prop_vowels <- str_count(words, "[aeiou]") / str_length(words)
words[which(prop_vowels == max(prop_vowels))]
#> [1] "a"
```

</div>

### Extract matches {#extract-matches .r4ds-section}

#### Exercise 14.4.2.1 {.unnumbered .exercise data-number="14.4.2.1"}

<div class="question">

In the previous example, you might have noticed that the regular expression matched “flickered”, which is not a color. 
Modify the regex to fix the problem.

</div>

<div class="answer">

This was the original color match pattern:

```r
colours <- c("red", "orange", "yellow", "green", "blue", "purple")
colour_match <- str_c(colours, collapse = "|")
```
It matches "flickered" because it matches "red".
The problem is that the previous pattern will match any word with the name of a color inside it. We want to only match colors in which the entire word is the name of the color.
We can do this by adding a `\b` (to indicate a word boundary) before and after the pattern:

```r
colour_match2 <- str_c("\\b(", str_c(colours, collapse = "|"), ")\\b")
colour_match2
#> [1] "\\b(red|orange|yellow|green|blue|purple)\\b"
```


```r
more2 <- sentences[str_count(sentences, colour_match) > 1]
```

```r
str_view_all(more2, colour_match2, match = TRUE)
```

<!--html_preserve--><div id="htmlwidget-c0b883f95c2bc807ca2a" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-c0b883f95c2bc807ca2a">{"x":{"html":"<ul>\n  <li>It is hard to erase <span class='match'>blue<\/span> or <span class='match'>red<\/span> ink.<\/li>\n  <li>The <span class='match'>green<\/span> light in the brown box flickered.<\/li>\n  <li>The sky in the west is tinged with <span class='match'>orange<\/span> <span class='match'>red<\/span>.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

</div>

#### Exercise 14.4.2.2 {.unnumbered .exercise data-number="14.4.2.2"}

<div class="question">

From the Harvard sentences data, extract:

1.  The first word from each sentence.
1.  All words ending in `ing`.
1.  All plurals.

</div>

<div class="answer">

The answer to each part follows.

1.  Finding the first word in each sentence requires defining what a pattern constitutes a word. For the purposes of this question,
    I'll consider a word any contiguous set of letters.

    
    ```r
    str_extract(sentences, "[a-zA-Z]+") %>% head()
    #> [1] "The"   "Glue"  "It"    "These" "Rice"  "The"
    ```

1.  This pattern finds all words ending in `ing`.

    
    ```r
    pattern <- "\\b[A-Za-z]+ing\\b"
    sentences_with_ing <- str_detect(sentences, pattern)
    unique(unlist(str_extract_all(sentences[sentences_with_ing], pattern))) %>%
      head()
    #> [1] "spring"  "evening" "morning" "winding" "living"  "king"
    ```

1.  Finding all plurals cannot be correctly accomplished with regular expressions alone.
    Finding plural words would at least require morphological information about words in the language.
    See [WordNet](https://cran.r-project.org/web/packages/wordnet/index.html) for a resource that would do that.
    However, identifying words that end in an "s" and with more than three characters, in order to remove "as", "is", "gas", etc., is
    a reasonable heuristic.

    
    ```r
    unique(unlist(str_extract_all(sentences, "\\b[A-Za-z]{3,}s\\b"))) %>%
      head()
    #> [1] "planks" "days"   "bowls"  "lemons" "makes"  "hogs"
    ```

</div>

### Grouped matches {#grouped-matches .r4ds-section}

#### Exercise 14.4.3.1 {.unnumbered .exercise data-number="14.4.3.1"}

<div class="question">

Find all words that come after a “number” like “one”, “two”, “three” etc. 
Pull out both the number and the word.

</div>

<div class="answer">

I'll use the same following "word" pattern as used above

```r
numword <- "(one|two|three|four|five|six|seven|eight|nine|ten) +(\\S+)"
sentences[str_detect(sentences, numword)] %>%
  str_extract(numword)
#>  [1] "ten served"    "one over"      "seven books"   "two met"      
#>  [5] "two factors"   "one and"       "three lists"   "seven is"     
#>  [9] "two when"      "one floor."    "ten inches."   "one with"     
#> [13] "one war"       "one button"    "six minutes."  "ten years"    
#> [17] "one in"        "ten chased"    "one like"      "two shares"   
#> [21] "two distinct"  "one costs"     "ten two"       "five robins." 
#> [25] "four kinds"    "one rang"      "ten him."      "three story"  
#> [29] "ten by"        "one wall."     "three inches"  "ten your"     
#> [33] "six comes"     "one before"    "three batches" "two leaves."
```

</div>

#### Exercise 14.4.3.2 {.unnumbered .exercise data-number="14.4.3.2"}

<div class="question">

Find all contractions.
Separate out the pieces before and after the apostrophe.

</div>

<div class="answer">


```r
contraction <- "([A-Za-z]+)'([A-Za-z]+)"
sentences[str_detect(sentences, contraction)] %>%
  str_extract(contraction)
#>  [1] "It's"       "man's"      "don't"      "store's"    "workmen's" 
#>  [6] "Let's"      "sun's"      "child's"    "king's"     "It's"      
#> [11] "don't"      "queen's"    "don't"      "pirate's"   "neighbor's"
```

</div>

### Replacing matches {#replacing-matches .r4ds-section}

#### Exercise 14.4.4.1 {.unnumbered .exercise data-number="14.4.4.1"}

<div class="question">
Replace all forward slashes in a string with backslashes.
</div>

<div class="answer">


```r
str_replace_all("past/present/future", "/", "\\\\")
#> [1] "past\\present\\future"
```

</div>

#### Exercise 14.4.4.2 {.unnumbered .exercise data-number="14.4.4.2"}

<div class="question">
Implement a simple version of `str_to_lower()` using `replace_all()`.
</div>

<div class="answer">

```r
replacements <- c(
  "A" = "a", "B" = "b", "C" = "c", "D" = "d", "E" = "e",
  "F" = "f", "G" = "g", "H" = "h", "I" = "i", "J" = "j",
  "K" = "k", "L" = "l", "M" = "m", "N" = "n", "O" = "o",
  "P" = "p", "Q" = "q", "R" = "r", "S" = "s", "T" = "t",
  "U" = "u", "V" = "v", "W" = "w", "X" = "x", "Y" = "y",
  "Z" = "z"
)
lower_words <- str_replace_all(words, pattern = replacements)
head(lower_words)
#> [1] "a"        "able"     "about"    "absolute" "accept"   "account"
```

</div>

#### Exercise 14.4.4.3 {.unnumbered .exercise data-number="14.4.4.3"}

<div class="question">
Switch the first and last letters in `words`. Which of those strings are still words?
</div>

<div class="answer">

First, make a vector of all the words with first and last letters swapped,

```r
swapped <- str_replace_all(words, "^([A-Za-z])(.*)([a-z])$", "\\3\\2\\1")
```
Next, find what of "swapped" is also in the original list using the function `intersect()`,

```r
intersect(swapped, words)
#>  [1] "a"          "america"    "area"       "dad"        "dead"      
#>  [6] "lead"       "read"       "depend"     "god"        "educate"   
#> [11] "else"       "encourage"  "engine"     "europe"     "evidence"  
#> [16] "example"    "excuse"     "exercise"   "expense"    "experience"
#> [21] "eye"        "dog"        "health"     "high"       "knock"     
#> [26] "deal"       "level"      "local"      "nation"     "on"        
#> [31] "non"        "no"         "rather"     "dear"       "refer"     
#> [36] "remember"   "serious"    "stairs"     "test"       "tonight"   
#> [41] "transport"  "treat"      "trust"      "window"     "yesterday"
```

</div>

### Splitting {#splitting .r4ds-section}

#### Exercise 14.4.5.1 {.unnumbered .exercise data-number="14.4.5.1"}

<div class="question">
Split up a string like `"apples, pears, and bananas"` into individual components.
</div>

<div class="answer">


```r
x <- c("apples, pears, and bananas")
str_split(x, ", +(and +)?")[[1]]
#> [1] "apples"  "pears"   "bananas"
```

</div>

#### Exercise 14.4.5.2 {.unnumbered .exercise data-number="14.4.5.2"}

<div class="question">
Why is it better to split up by `boundary("word")` than `" "`?
</div>

<div class="answer">

Splitting by `boundary("word")` is a more sophisticated method to split a string into words.
It recognizes non-space punctuation that splits words, and also removes punctuation while retaining internal non-letter characters that are parts of the word, e.g., "can't"
See the [ICU website](http://userguide.icu-project.org/boundaryanalysis) for a description of the set of rules that are used to determine word boundaries.

Consider this sentence from the official [Unicode Report on word boundaries](http://www.unicode.org/reports/tr29/#Word_Boundaries),

```r
sentence <- "The quick (“brown”) fox can’t jump 32.3 feet, right?"
```
Splitting the string on spaces considers will group the punctuation with the words,

```r
str_split(sentence, " ")
#> [[1]]
#> [1] "The"       "quick"     "(“brown”)" "fox"       "can’t"     "jump"     
#> [7] "32.3"      "feet,"     "right?"
```
However, splitting the string using `boundary("word")` correctly removes punctuation, while not
separating "32.2" and "can't",

```r
str_split(sentence, boundary("word"))
#> [[1]]
#> [1] "The"   "quick" "brown" "fox"   "can’t" "jump"  "32.3"  "feet"  "right"
```

</div>

#### Exercise 14.4.5.3 {.unnumbered .exercise data-number="14.4.5.3"}

<div class="question">
What does splitting with an empty string `("")` do? Experiment, and then read the documentation.
</div>

<div class="answer">


```r
str_split("ab. cd|agt", "")[[1]]
#>  [1] "a" "b" "." " " "c" "d" "|" "a" "g" "t"
```

It splits the string into individual characters.

</div>

### Find matches {#find-matches .r4ds-section}

<!--html_preserve--><div class="alert alert-warning hints-alert">
<div class="hints-icon">
<i class="fa fa-exclamation-circle"></i>
</div>
<div class="hints-container">No exercises</div>
</div><!--/html_preserve-->

## Other types of pattern {#other-types-of-pattern .r4ds-section}

### Exercise 14.5.1 {.unnumbered .exercise data-number="14.5.1"}

<div class="question">
How would you find all strings containing `\` with `regex()` vs. with `fixed()`?
</div>

<div class="answer">


```r
str_subset(c("a\\b", "ab"), "\\\\")
#> [1] "a\\b"
str_subset(c("a\\b", "ab"), fixed("\\"))
#> [1] "a\\b"
```

</div>

### Exercise 14.5.2 {.unnumbered .exercise data-number="14.5.2"}

<div class="question">

What are the five most common words in `sentences`?

</div>

<div class="answer">

Using `str_extract_all()` with the argument `boundary("word")` will extract all words.
The rest of the code uses dplyr functions to count words and find the most
common words.

```r
tibble(word = unlist(str_extract_all(sentences, boundary("word")))) %>%
  mutate(word = str_to_lower(word)) %>%
  count(word, sort = TRUE) %>%
  head(5)
#> # A tibble: 5 x 2
#>   word      n
#>   <chr> <int>
#> 1 the     751
#> 2 a       202
#> 3 of      132
#> 4 to      123
#> 5 and     118
```

</div>

## Other uses of regular expressions {#other-uses-of-regular-expressions .r4ds-section}

<!--html_preserve--><div class="alert alert-warning hints-alert">
<div class="hints-icon">
<i class="fa fa-exclamation-circle"></i>
</div>
<div class="hints-container">No exercises</div>
</div><!--/html_preserve-->

## stringi {#stringi .r4ds-section}


```r
library("stringi")
```

### Exercise 14.7.1 {.unnumbered .exercise data-number="14.7.1"}

<div class="question">

Find the stringi functions that:

1.  Count the number of words.
1.  Find duplicated strings.
1.  Generate random text.

</div>

<div class="answer">

The answer to each part follows.

1.  To count the number of words use `stringi::stri_count_words()`.
    This code counts the words in the first five sentences of `sentences`.
    
    ```r
    stri_count_words(head(sentences))
    #> [1] 8 8 9 9 7 7
    ```

1.  The `stringi::stri_duplicated()` function finds duplicate strings.
    
    ```r
    stri_duplicated(c(
      "the", "brown", "cow", "jumped", "over",
      "the", "lazy", "fox"
    ))
    #> [1] FALSE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE
    ```

1.  The *stringi* package contains several functions beginning with `stri_rand_*` that generate random text.
    The function `stringi::stri_rand_strings()` generates random strings.
    The following code generates four random strings each of length five.
    
    ```r
    stri_rand_strings(4, 5)
    #> [1] "5pb90" "SUHjl" "sA2JO" "CP3Oy"
    ```
    
    The function `stringi::stri_rand_shuffle()` randomly shuffles the characters in the text.
    
    ```r
    stri_rand_shuffle("The brown fox jumped over the lazy cow.")
    #> [1] "ot f.lween p   jzwoom xyucobhv daheerrT"
    ```
    
    The function `stringi::stri_rand_lipsum()` generates [lorem ipsum](https://en.wikipedia.org/wiki/Lorem_ipsum) text.
    Lorem ipsum text is nonsense text often used as placeholder text in publishing.
    The following code generates one paragraph of placeholder text.
    
    ```r
    stri_rand_lipsum(1)
    #> [1] "Lorem ipsum dolor sit amet, hac non metus cras nam vitae tempus proin, sed. Diam gravida viverra eros mauris, magna lacinia dui nullam. Arcu proin aenean fringilla sed sollicitudin hac neque, egestas condimentum massa, elementum vivamus. Odio eget litora molestie eget eros pulvinar ac. Vel nec nullam vivamus, sociosqu lectus varius eleifend. Vitae in. Conubia ut hac maximus amet, conubia sed. Volutpat vitae class cursus, elit mauris porta. Mauris lacus donec odio eget quam inceptos, ridiculus cursus, ad massa. Rhoncus hac aenean at id consectetur molestie vitae! Sed, primis mi dictum lacinia eros. Ligula, feugiat consequat ut vivamus ut morbi et. Dolor, eget eleifend nec magnis aliquam egestas. Sollicitudin venenatis et aptent rhoncus nisl platea ligula cum."
    ```

</div>

### Exercise 14.7.2 {.unnumbered .exercise data-number="14.7.2"}

<div class="question">

How do you control the language that `stri_sort()` uses for sorting?

</div>

<div class="answer">

You can set a locale to use when sorting with either `stri_sort(..., opts_collator=stri_opts_collator(locale = ...))` or `stri_sort(..., locale = ...)`.
In this example from the `stri_sort()` documentation, the sorted order of the character vector depends on the locale.

```r
string1 <- c("hladny", "chladny")
stri_sort(string1, locale = "pl_PL")
#> [1] "chladny" "hladny"
stri_sort(string1, locale = "sk_SK")
#> [1] "hladny"  "chladny"
```

The output of `stri_opts_collator()` can also be used for the `locale` argument of `str_sort`.

```r
stri_sort(string1, opts_collator = stri_opts_collator(locale = "pl_PL"))
#> [1] "chladny" "hladny"
stri_sort(string1, opts_collator = stri_opts_collator(locale = "sk_SK"))
#> [1] "hladny"  "chladny"
```
The `stri_opts_collator()` provides finer grained control over how strings are sorted.
In addition to setting the locale, it has options to customize how cases, unicode, accents, and numeric values are handled when comparing strings.

```r
string2 <- c("number100", "number2")
stri_sort(string2)
#> [1] "number100" "number2"
stri_sort(string2, opts_collator = stri_opts_collator(numeric = TRUE))
#> [1] "number2"   "number100"
```

</div>
