## Introduction

The University of Wisconsin, Madison (UW-Madison) has [public
data](https://registrar.wisc.edu/grade-reports/) of undergraduate
grade-point averages, as well as undergraduate grade-distribution
reports, which data spans from 2014-2015 to 2021-2022. Unfortunately,
the data is stored in PDFs, which is convenient for reading but less so
for data analysis, so I have only extracted the aggregate information of
GPA, excluding grade distributions, school-specific data, and
program-specific data. Further data extraction would require an
automated reader for PDF…or determination. The goal of this project is
to analyze said aggregate information of GPA. As a last note, you can
also find [public data with a
dashboard](https://registrar.wisc.edu/enrollment-reports/) of enrollment
reports for UW-Madison from 2010-2011 to 2019-2020.

## Data

As stated, the UW-Madison undergraduate reports have more information
regarding GPA, but I only extracted (1) student population counts and
(2) average GPA, both with sub-categories of academic class (i.e.,
freshman, sophomore, junior, senior) and identification (i.e., men and
women). Let’s examine the data:

    madgpa <- read_csv("/Users/maxwellhanson/Documents/R/uwmad-gpa/uwmad-gpa.csv")

    ## Rows: 192 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (5): id, academic_year, semester, class, season
    ## dbl (3): year, count, gpa
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    spec(madgpa)

    ## cols(
    ##   id = col_character(),
    ##   academic_year = col_character(),
    ##   year = col_double(),
    ##   semester = col_character(),
    ##   class = col_character(),
    ##   count = col_double(),
    ##   gpa = col_double(),
    ##   season = col_character()
    ## )

    madgpa %>% head(n = 12)

    ## # A tibble: 12 × 8
    ##    id    academic_year  year semester  class     count   gpa season
    ##    <chr> <chr>         <dbl> <chr>     <chr>     <dbl> <dbl> <chr> 
    ##  1 total 2014-2015      2014 Fall 2014 freshman   4863  3.16 fall  
    ##  2 total 2014-2015      2014 Fall 2014 sophomore  6185  3.16 fall  
    ##  3 total 2014-2015      2014 Fall 2014 junior     7314  3.24 fall  
    ##  4 total 2014-2015      2014 Fall 2014 senior    10260  3.35 fall  
    ##  5 men   2014-2015      2014 Fall 2014 freshman   2231  3.10 fall  
    ##  6 men   2014-2015      2014 Fall 2014 sophomore  3052  3.10 fall  
    ##  7 men   2014-2015      2014 Fall 2014 junior     3587  3.16 fall  
    ##  8 men   2014-2015      2014 Fall 2014 senior     5123  3.25 fall  
    ##  9 women 2014-2015      2014 Fall 2014 freshman   2632  3.21 fall  
    ## 10 women 2014-2015      2014 Fall 2014 sophomore  3133  3.21 fall  
    ## 11 women 2014-2015      2014 Fall 2014 junior     3727  3.31 fall  
    ## 12 women 2014-2015      2014 Fall 2014 senior     5137  3.45 fall

I include `season` because I would like to make comparisons between Fall
and Spring, and it’s easier to sort `season` over `semester`.

## GPA over Time

Naturally, we examine the fluctuations of GPA over time, splitting the
data into `men`, `women`, and `total`.

    # the following line is added to format plot legends
    madgpa$class <- factor(madgpa$class, levels = c('freshman', 'sophomore', 'junior', 'senior'))

    madgpa %>% 
      filter(id == 'men') %>% 
      arrange(year) %>% 
      ggplot(aes(x = semester, y = gpa, color = class)) +
      geom_point() +
      scale_x_discrete(limits = unique(madgpa$semester)) +
      scale_y_continuous(n.breaks = 12) +
      theme(axis.text.x = element_text(angle = 90), axis.title.x = element_blank()) +
      xlab('Semester and Year') +
      ylab('GPA') +
      ggtitle('GPA of Men at UW-Madison by Semester (2014-2022)') +
      labs(color = 'Class')

![](uwmad-gpa_files/figure-markdown_strict/men-class-gpa-1.png)

    madgpa %>% 
      filter(id == 'women') %>% 
      arrange(year) %>% 
      ggplot(aes(x = semester, y = gpa, color = class)) +
      geom_point() +
      scale_x_discrete(limits = unique(madgpa$semester)) +
      scale_y_continuous(n.breaks = 12) +
      theme(axis.text.x = element_text(angle = 90),
            axis.title.x = element_blank()) +
      xlab('Semester and Year') +
      ylab('GPA') +
      ggtitle('GPA of Women at UW-Madison by Semester (2014-2022)') +
      labs(color = 'Class')

![](uwmad-gpa_files/figure-markdown_strict/women-class-gpa-1.png)

    madgpa %>% 
      filter(id == 'total') %>% 
      arrange(year) %>% 
      ggplot(aes(x = semester, y = gpa, color = class)) +
      geom_point() +
      scale_x_discrete(limits = unique(madgpa$semester)) +
      scale_y_continuous(n.breaks = 12) +
      theme(axis.text.x = element_text(angle = 90),
            axis.title.x = element_blank()) +
      xlab('Semester and Year') +
      ylab('GPA') +
      ggtitle('Total GPA at UW-Madison by Semester (2014-2022)') +
      labs(color = 'Class')

![](uwmad-gpa_files/figure-markdown_strict/total-class-gpa-1.png)

There are some interesting observations about GPA from the graphs:

-   There is a general upward trend in GPA over time for each class,
    which we could deem “grade inflation”, but the label isn’t nuanced.
    We would want more information about grade distribution, for
    example. (See: Introduction.)
-   Related to the upward trend in GPA over time, we see a large jump in
    GPA beginning Spring 2020, when UW-Madison faced COVID. In fact,
    UW-Madison offered a Pass/Fail (P/F) option during the Spring 2020
    semester (and only the Spring 2020 semester), but it seems there was
    some inertia with sympathetic grading. (Note: I didn’t opt for
    Pass/Fail.)
-   There is a strange oscillatory behavior in GPA for freshmen between
    Fall and Spring. Actually, we observe an oscillatory behavior in the
    freshmen student population, too (see: next graph). Is the cause
    freshmen struggling to adjust to college?

## Student Population

We expect the student population to be relatively consistent between
semesters for a given academic year, but, in fact, we observe that there
is a major drop-off in the freshmen student population from Fall to
Spring for any academic year:

    madgpa %>% 
      filter(id == 'total') %>% 
      arrange(year) %>% 
      ggplot(aes(x = semester, y = count, color = class)) +
      geom_point() +
      scale_x_discrete(limits = unique(madgpa$semester)) +
      scale_y_continuous(n.breaks = 12) +
      theme(axis.text.x = element_text(angle = 90)) +
      xlab('Semester and Year') +
      ylab('Count') +
      ggtitle('Student Population at UW-Madison by Class (2014-2022)') +
      labs(color = 'Class')

![](uwmad-gpa_files/figure-markdown_strict/total-count-gpa-1.png)

On average, by how much does the freshmen student population fall from
Fall to Spring in an academic year? Ans: 3100.

    freshmen_drop <- (madgpa %>% 
       filter(id == 'total' & class == 'freshman' & season == 'fall') %>% 
       select(count)) - 
    (madgpa %>% 
       filter(id == 'total' & class == 'freshman' & season == 'spring') %>% 
       select(count))

    mean(freshmen_drop$count)

    ## [1] 3100

## GPA: Men versus Women

Another interesting point of analysis is the differences between men and
women in GPA. We find that the women dominate:

    madgpa %>% 
      filter(id != 'total') %>%
      ggplot(aes(x = id, y = gpa, fill = id)) +
      geom_boxplot() +
      theme(axis.title.x = element_blank()) +
      ylab('GPA') +
      ggtitle('GPA for Men and Women at UW-Madison by Class (2014-2022)') +
      facet_wrap(~class) +
      scale_x_discrete(labels = c('Men', 'Women')) +
      labs(fill = 'ID')

![](uwmad-gpa_files/figure-markdown_strict/men-women-box-1.png)

## Conclusion

Again, there’s a lot more data, and there’s a lot more to do with the
data, but reading and formatting the data is its own project. (See:
Introduction.) Of course, with more data, it would be interesting to
examine GPA according to school and program at UW-Madison. Here, I’ve
analyzed student population and GPA according to academic class and
identity, and besides more tinkering, that is the capacity of this data.
We observe “grade inflation” from 2014-2015 to 2021-2022, with a spike
due to COVID. Moreover, women dominate men when it comes to GPA, and we
observe some oscillatory behavior in student population for freshmen and
seniors, especially for freshmen, whose population drops on average
3,100 from Fall to Spring. I hope to revisit this project in the future
to add more data and analysis.