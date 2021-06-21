# The funkyfigs package

To see what package building was all about, I created the `funkyfigs` package. This package, for now, consists of four functions, designed to make data wrangling and visualisation a little easier.

* There is the `blast_graph` function, which gives a clear overview graph on the composition after you've run a sample in BLAST+.  

* The `photospectro_bar` and `photospectro_boxplot` functions make it a little easier to create bar graps and boxplots, respectively, for photospectrometry measurements (because we all know that putting in those measurement is enough work on it's own). They can also be used for other data though! Just don't put it in any official reports as the labs will be designed for photospectrography.    

* Finally, there is the `url_table` function, that will add a column with individual url links per cell to your table/tibble/dataframe/etc., so you can easily keep additional information close by!

Curious? You can install the package for yourself, using [this Github repository](https://github.com/IreneSchimmel/funkyfigs)!   
If you want any more information on the package or it's functions, I've made a (I hope) very clear vignette. This is also available in the repository mentioned above.
