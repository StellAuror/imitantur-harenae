---
title: "Machine Learning Fundamentals"
date: 2021-06-08
#hero: /images/posts/writing-posts/analytics.svg
description: Basic statistics and measures for machine learning algorithms 
theme: Toha
mermaid: true
---

```{r include=FALSE}
library("knitr")
```

## Intro

### Biblioteki

```{r intro,  message=FALSE, warning=FALSE}
library("tidyverse")
library("plotly")
library("gapminder")
library("broom")
library("ggfortify")
```

### Model liniowy `lm()`

```{r echo=TRUE}
# Wszystkie zmienne
model <- 
  filter(gapminder, country == "Poland")[, -(1:2)] %>%
    lm(formula = pop ~ ., data = .)

```

```{r echo=FALSE, warning=FALSE}
years <- filter(gapminder, country == "Poland")$year

 filter(gapminder, country == "Poland") %>%
   plot_ly(
     x = ~year,
     y = ~pop
   ) %>% add_markers(name = "Dane historyczne") %>% 
   add_lines(color = "red", x = years, y = fitted(model), name = "Model")


```

-   Model liniowy

-   Dotyczy tylko zmiennych numerycznych

-   funkcja **I()** pozwala na modyfikację zmiennych (np. potęga) w celu modyfikacji charakterystyki rozkładu zmiennej do modelu liniowego

    -   np. jeżeli wykres rozrzutu ma wzór paraboli, zmienną objaśnianą należy podnieść do potęgi $^{\frac{1}{2}}$

## Funkcje tekstowe
### Summary

```{r}
summary(model)
```
### Augment
```{r echo=T, results='hide'}
augment(model)
```

```{r echo=FALSE}
kable(augment(model))
```
### Glance
```{r echo=T, results='hide'}
glance(model)
```

```{r echo=FALSE}
kable(glance(model))
```

## Funkcje graficzne

### Residuals vs. Fitted

```{r}
autoplot(
    model,
    which = 1, nrow = 1, ncol = 1
)

```

-   Pytanie: **Jaki jest trend poprawności modelu**

-   Im residuals bliższe są 0, tym lepszy model

-   Pokazuje czy poszczególne przedziały modelu charakteryzują się różnymi trendami

### QQ Plot

```{r}
autoplot(
    model,
    which = 2, nrow = 1, ncol = 1
)

```

-   Pytanie: **Czy model załamuje się w specyficznych miejscach?**

-   Oś y - zestandaryzowane wartości reszt (residuals)

-   Oś x - kwantyle zmiennej objaśniącej

### Scale - location

```{r}
autoplot(
    model,
    which = 3, nrow = 1, ncol = 1
)

```

-   Pytanie: **Czy trend poprawności modelu rośnie, czy też spada?**

-   Podobny do *resudials vs. fitted values*, ale w tym wypadku *residuals* są pierwiastkiem ze zestandaryzowanych wartości

## Statystyki

### Coefficients

```{r}
coefficients(model)
```

-   **Współczynniki funkcji liniowej**

    -   Pierwsza wartość - wyraz wolny *intercept*

    -   Druga wartość - współczynnik kierunkowy *slope*

### Fitted values

```{r}
fitted(model)
```

```{r echo=FALSE}
x <- fitted(model)
names(x) <-  years
x
```

-   Odpowiada na pytanie: **Jakie wartości predykuje model?**

-   Wektor wartości estymowanych wg podanego modelu

### Residuals

```{r}
residuals(model)
```

-   Wektor błędu estymacji

    -   Formuła: $wartość - estymacja$

-   Pytanie: **Jak bardzo estymacje różnią się od faktycznych wartości?**

### R-squared

```{r}
glance(model) %>% pull(r.squared)
```

-   Odpowiada na pytanie: **Jak dobry jest model?**

-   Jest to wartość korelacji podniesiona do kwadratu, więc:

    -   1 - perfekcyjne dopasowanie

    -   0 - fatalne dopasowanie

### Sigma / RSE / RSME

```{r}
glance(model) %>% pull(sigma) #RSE
```

-   Pytanie: **Jaki jest przeciętny błąd modelu (między obserwacją, a predykcją)?**

-   Wzór\$ RSE = \frac {\\sum Residuals\^2} {freedom Degree} \^ {1/2}\$

    -   Freedom degree - liczba obserwacji - liczba współczynników

    -   Przy obliczaniu RMSE należy pominąć liczbę współczynników *(number of coefficients)*

### Leverage / Hat values

```{r}
hatvalues(model)
```

-   Pytanie: **Jak bardzo skrajne są poszczególne estymacje**

-   Im niższa wartość, tym bardziej skrajna estymacja

### Influence (Cook's distance)

```{r message=FALSE, warning=FALSE}

model %>%
  augment %>%
  select(.cooksd)
```

```{r echo=FALSE}
kable(model %>%
  augment %>%
  select(.cooksd))
```

-   Pytanie: ***Jak bardzo model zmieniłbym się, jeżeli poszczególne obserwacje zostałyby pominięte, tudzież jaki wpływ ma obserwacja na model***

-   Im wyższa wartość, tym bardziej skrajna obserwacja

# Tidyverse overview
## Wstęp
### Biblioteki

```{r message=FALSE, warning=FALSE}
library("tidyverse")
library("plotly")
library("gapminder")
```

### Zbiór danych `gapminder`

```{r}
kable(head(gapminder),
      col.names = names(gapminder),
      align = "cc",
      caption = "Pierwsze 6 rekordów zbioru")

```

## Funkcje `nest()` oraz `unnest()`

```{r echo=T, message=FALSE, warning=FALSE, results='hide'}
gapminder %>% 
  group_by(country) %>% 
  nest()

# dfN %>% unnest()
```

```{r echo=FALSE}
dfN <- 
  gapminder %>% 
  group_by(country) %>% 
  nest()


kable(head(dfN, 1),
      col.names = names(dfN)[1:2],
      align = "cc")

kable(head(dfN$data[[1]]), 
      align = "cc")
```

## Mapowanie `map()`

### Z funkcją `mutate()`

`map()` przyjmuje parametr `data`, który jest kolumną (tibble) struktury danych `dfN`

Jeżeli outputem jest jedna zmienna (tj. wektor o długości 1), to można uprościć jej wyświetlanie za pomocą rodziny funkcji `map_*()`

```{r}
  dfN %>%
  mutate(mean_pop = map(data, ~mean(.x$pop)))
  
  output <-
    dfN %>%
    mutate(mean_pop = map_dbl(data, ~mean(.x$pop)))
```

```{r include=FALSE}
kable(head(output, 1))
```

### Z modelem liniowym `lm()`

```{r}
dfNm <- 
  dfN %>%
  mutate(model = map(data, ~lm(formula = pop ~ ., data = .x[, 2:4])))

output <- 
  dfNm %>% 
    mutate(perc_bias = map2_dbl(data, model, ~mean(.y$residuals / .x$pop)))

```

```{r echo=FALSE, warning=FALSE}
output %>% 
  unnest(data) %>%
  group_by(continent, country, perc_bias) %>%
  summarise() %>%
  group_by(continent) %>%
  do(p  =
    plot_ly(.,
      x = ~perc_bias,
      y = ~country,
      name = ~continent
    ) %>%
    add_bars() %>%
    layout(
      xaxis = list(title = ""),
      yaxis = list(title = "",
                   tickvals  = NA)
    )
  ) %>% subplot(nrows = 3, shareX = T, shareY = F)
```
