---
title: "Projekt z analizy danych"
author: "Filip Nienartowicz 122531"
date: "09 grudzie� 2018"
output: 
  html_document:
    keep_md: yes
    toc: yes
    toc_float: yes
---



#Podsumowanie

(poni�sze komentarze dost�pne s� r�wnie� w miejscu ich dotycz�cym)
##a.d.3. Kod pozwalaj�cy wczyta� dane z pliku.
Do wczytania danych wykorzysta�em funkcj� fread z pakietu data.table. Funkcja ta jest znacznie szybsza ni� read.table i pozwala na usuwanie zb�dnych kolumn ju� na etapie wczytywania danych. Dodatkowo prawid�owo rozpoznaje kolumny o typie Integer
##a.d.5. Kod przetwarzaj�cy brakuj�ce dane.
Dane zostaj� podzielone na 3 cz�ci (part_00, part_01 i part_02). Prefisky kolumn usuni�te, dodana kolumna part z warto�ci� odpowiedniego partu b�d�cego �r�d�em danych. Nast�pnie dane s� ��czone w jeden data frame. Proces ten usuwa wszystkie warto�ci NA - uzyskane dane s� wi�c wyczyszczone.

Je�eli any_meas* = 0, to zak�adam, �e nie nast�pi�� pomiar (w kolejnych kolumnach dane NA, gdy� nie by�o podstawy do oblicze�). W takiej sytuacji zamiana NA na np. �redni� nie ma sensu (0 elektron�w nie mo�e mie� kszta�tu itp.) dlatego te� dane dotycz�ce danego part zostaj� usuni�te.

*any_meas = shape_segments_count + density_segments_count + volume + electrons + mean + std + max + max_over_std + skewness + parts) [�adna z tych zmiennych nie przyjmuje warto�ci < 0, wi�c suma == 0 jednoznacznie wskazuje na brak tych danych]

##a.d.8. Sekcj� sprawdzaj�c� korelacje mi�dzy zmiennymi
Poni�ej znajduje si� wykres korelacji mi�dzy zmiennymi. Zauwa�y� mo�na, �e:
* Du�a cz�� zmiennych jest w silnej korelacji z innymi zmiennymi
* Zmienne density_Z* i shape_z* s� w bardzo silnej korelacji (du�e czerwone prostok�ty)
* Zmienne local* koreluj� z zmiennymi z part (by�y obliczane na ich podstawie)

##a.d.14 i 15
Spo�r�d testowanych metod najlepiej sprawdzi�y si�:
* Linear Regression (lm) dla regresji
* Random Forest (rf) dla klasyfikacji

Z racji du�ego zbioru danych (ok. 1 mln rekord�w) za metod� wybierania zbioru walidacyjnego wybra�em bootstrap (losowanie ze zwracaniem) - metoda zapewnia stratyfikacj� danych (https://machinelearningmastery.com/how-to-estimate-model-accuracy-in-r-using-the-caret-package/)

##a.d.14.Sekcj� sprawdzaj�c� czy na podstawie warto�ci innych kolumn mo�na przewidzie�:
Na podstawie oblicze� usatli�em, �e odpowiednio 3% i 1% danych powinien wystarczy� jako zbi�r testowy (z obliczeniami mo�na zapozna� si� w sekcji 14)

Liczba elektron�w zosta�a okre�lona z miarami*:
RMSE = 63,835
r^2 = 0,49


Liczba elektron�w zosta�a okre�lona z miarami*:
RMSE = 9,544
r^2 = 0,48

##a.d.15.Sekcj� pr�buj�c� stworzy� klasyfikator przewiduj�cy warto�� atrybutu res_name 
Na podstawie oblicze� usatli�em, �e 1% danych powinien wystarczy� jako zbi�r testowy (z obliczeniami mo�na zapozna� si� w sekcji 15)

Random Forest uzyska� precyzj�* = n%

Klasyfikator naiwny (wskazuj�cy najliczniejsz� klas�) dokona�by predykcji res_name = SO4. Miara Accuracy = 15%.

* Podane warto�ci mog� nie by� zgodne z ko�cowymi wynikami. Z rezultatem mo�na zapozna� si� w cz�ci 14 i 15 niniejszego raportu.

# 1. Kod wyliczaj�cy wykorzystane biblioteki.

```r
library(data.table)
library(DT)
library(ggplot2)
library(plotly)
library(dplyr)
library(reshape)
library(caret)
```



# 2. Kod zapewniaj�cy powtarzalno�� wynik�w.

```r
set.seed(123)
```

# 3. Kod pozwalaj�cy wczyta� dane z pliku.
Do wczytania danych wykorzysta�em funkcj� fread z pakietu data.table. Funkcja ta jest znacznie szybsza ni� read.table i pozwala na usuwanie zb�dnych kolumn ju� na etapie wczytywania danych. Dodatkowo prawid�owo rozpoznaje kolumny o typie Integer 

```r
out_columns <- c("blob_coverage", "res_coverage", "title", "pdb_code", "res_id", "chain_id", "blob_volume_coverage", "blob_volume_coverage_second", "res_volume_coverage", "res_volume_coverage_second", "skeleton_cycle_4", "skeleton_diameter", "skeleton_cycle_6", "skeleton_cycle_7", "skeleton_closeness_006_008", "skeleton_closeness_002_004", "skeleton_cycle_3", "skeleton_avg_degree", "skeleton_closeness_004_006", "skeleton_closeness_010_012", "skeleton_closeness_012_014", "skeleton_edges", "skeleton_radius", "skeleton_cycle_8_plus", "skeleton_closeness_020_030", "skeleton_deg_5_plus", "skeleton_closeness_016_018", "skeleton_closeness_008_010", "skeleton_closeness_018_020", "skeleton_average_clustering", "skeleton_closeness_040_050", "skeleton_closeness_014_016", "skeleton_center", "skeleton_closeness_000_002", "skeleton_density", "skeleton_closeness_030_040", "skeleton_deg_4", "skeleton_deg_0", "skeleton_deg_1", "skeleton_deg_2", "skeleton_deg_3", "skeleton_graph_clique_number", "skeleton_nodes", "skeleton_cycles", "skeleton_cycle_5", "skeleton_closeness_050_plus", "skeleton_periphery", "local_cut_by_mainchain_volume", "local_near_cut_count_C", "local_near_cut_count_other", "local_near_cut_count_S", "local_near_cut_count_O", "local_near_cut_count_N", "fo_col", "fc_col", "weight_col", "grid_space", "solvent_radius", "solvent_opening_radius", "resolution_max_limit", "part_step_FoFc_std_min", "part_step_FoFc_std_max", "part_step_FoFc_std_step", "skeleton_data", "local_res_atom_count", "local_res_atom_non_h_occupancy_sum", "local_res_atom_non_h_electron_occupancy_sum", "local_res_atom_C_count", "local_res_atom_N_count", "local_res_atom_O_count", "local_res_atom_S_count", "dict_atom_C_count", "dict_atom_N_count", "dict_atom_O_count", "dict_atom_S_count")

file = "C:/Users/Developer/Desktop/all_summary.csv"
data <- fread(file = file, 
             sep = ";", 
             header = TRUE,
             na.string = c(",,", "NAN", "nan"),
             drop = out_columns
            )
```

# 4. Kod usuwaj�cy z danych wiersze posiadaj�ce warto�� zmiennej res_name

```r
out_res_names = c("UNK", "UNX", "UNL", "DUM", "N", "BLOB", "ALA", "ARG", "ASN", "ASP", "CYS", "GLN", "GLU", "GLY", "HIS", "ILE", "LEU", "LYS", "MET", "MSE", "PHE", "PRO", "SEC", "SER", "THR", "TRP", "TYR", "VAL", "DA", "DG", "DT", "DC", "DU", "A", "G", "T", "C", "U", "HOH", "H20", "WAT")
data <- data %>% filter(!res_name %in% out_res_names)
```

# 5. Kod przetwarzaj�cy brakuj�ce dane.
Dane zostaj� podzielone na 3 cz�ci (part_00, part_01 i part_02). Prefisky kolumn usuni�te, dodana kolumna part z warto�ci� odpowiedniego partu b�d�cego �r�d�em danych. Nast�pnie dane s� ��czone w jeden data frame. Proces ten usuwa wszystkie warto�ci NA - uzyskane dane s� wi�c wyczyszczone.

Je�eli any_meas* = 0, to zak�adam, �e nie nast�pi�� pomiar (w kolejnych kolumnach dane NA, gdy� nie by�o podstawy do oblicze�). W takiej sytuacji zamiana NA na np. �redni� nie ma sensu (0 elektron�w nie mo�e mie� kszta�tu itp.) dlatego te� dane dotycz�ce danego part zostaj� usuni�te.

*any_meas = shape_segments_count + density_segments_count + volume + electrons + mean + std + max + max_over_std + skewness + parts) [�adna z tych zmiennych nie przyjmuje warto�ci < 0, wi�c suma == 0 jednoznacznie wskazuje na brak tych danych]


```r
data_part00 <- data %>% 
    select(-starts_with("part_01"), -starts_with("part_02")) %>% 
    rename_at(.vars = vars(starts_with("part_00_")), .funs = funs(sub("^part_00_", "", .))) %>%
    mutate(any_meas = shape_segments_count + density_segments_count + volume + electrons + mean + std + max + max_over_std + skewness + parts) %>%       
    filter(any_meas != 0) %>% 
    select(-any_meas) %>%
    mutate(part = "part_00")

data_part01 <- data %>% 
    select(-starts_with("part_00"), -starts_with("part_02")) %>% 
    rename_at(.vars = vars(starts_with("part_01_")), .funs = funs(sub("^part_01_", "", .))) %>%
    mutate(any_meas = shape_segments_count + density_segments_count + volume + electrons + mean + std + max + max_over_std + skewness + parts) %>%       
    filter(any_meas != 0) %>% 
    select(-any_meas) %>%  
    mutate(part = "part_01")

data_part02 <- data %>% 
    select(-starts_with("part_00"), -starts_with("part_01")) %>% 
    rename_at(.vars = vars(starts_with("part_02_")), .funs = funs(sub("^part_02_", "", .))) %>%
    mutate(any_meas = shape_segments_count + density_segments_count + volume + electrons + mean + std + max + max_over_std + skewness + parts) %>%       
    filter(any_meas != 0) %>% 
    select(-any_meas) %>%  
    mutate(part = "part_02")
data <- rbind(data_part00, data_part01, data_part02) 
```
W kolumnach znajduje si� local_min - kolumna ta ma 0 we wszystkich wierszach - jest wi�c zb�dna

```r
data %>% select(local_min) %>% distinct()
```

```
##   local_min
## 1         0
```

```r
data <- data %>% select(-local_min)
rm(data_part00, data_part01, data_part02)
```

# 6. Sekcj� podsumowuj�c� rozmiar zbioru i podstawowe statystyki.
##Wymiary:

```r
dim(data)
```

```
## [1] 1710450     125
```

##Liczba unikalnych res_name

```r
data %>% select(res_name) %>% distinct() %>% nrow()
```

```
## [1] 19603
```

##Przyk�ad�w/part

```r
data %>% group_by(part) %>% summarize(n = n()) %>% arrange(desc(n)) %>% prettyTable()
```

<!--html_preserve--><div id="htmlwidget-49c968e2f00b87e48d74" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-49c968e2f00b87e48d74">{"x":{"style":"bootstrap","filter":"none","data":[["part_00","part_01","part_02"],[585330,579844,545276]],"container":"<table class=\"table table-striped table-hover\">\n  <thead>\n    <tr>\n      <th>part<\/th>\n      <th>n<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"dom":"Bfrtip","buttons":["copy","csv","excel","pdf","print"],"scrollX":true,"columnDefs":[{"className":"dt-right","targets":1}],"order":[],"autoWidth":false,"orderClasses":false,"rowCallback":"function(row, data) {\n}"}},"evals":["options.rowCallback"],"jsHooks":[]}</script><!--/html_preserve-->

##Podstawowe statystyki pozosta�ych kolumn

```r
data %>% select(-res_name, -starts_with("part")) %>%
  summary() %>% 
  unclass() %>%
  data.frame(check.names = FALSE, stringsAsFactors = FALSE) %>% 
  prettyTable()
```

<!--html_preserve--><div id="htmlwidget-f474ad921ae63e0a53f4" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-f474ad921ae63e0a53f4">{"x":{"style":"bootstrap","filter":"none","data":[["Min.   :  1.0  ","1st Qu.:  4.0  ","Median :  6.0  ","Mean   : 13.4  ","3rd Qu.: 19.0  ","Max.   :111.0  ",null],["Min.   :   3.00  ","1st Qu.:  30.00  ","Median :  48.00  ","Mean   :  99.63  ","3rd Qu.: 132.00  ","Max.   :1848.00  ",null],["Min.   :  1.00  ","1st Qu.:  4.00  ","Median :  6.00  ","Mean   : 13.69  ","3rd Qu.: 19.00  ","Max.   :128.00  ","NA's   :32268  "],["Min.   :   3.0  ","1st Qu.:  30.0  ","Median :  48.0  ","Mean   : 101.8  ","3rd Qu.: 132.0  ","Max.   :1223.0  ","NA's   :32268  "],["Min.   :   57.76  ","1st Qu.:  215.49  ","Median :  351.65  ","Mean   :  868.86  ","3rd Qu.:  807.84  ","Max.   :90952.51  ",null],["Min.   :  0.0149  ","1st Qu.:  3.6652  ","Median :  8.0314  ","Mean   : 18.0497  ","3rd Qu.: 20.1175  ","Max.   :442.4445  ",null],["Min.   :0.0001147  ","1st Qu.:0.0125222  ","Median :0.0189848  ","Mean   :0.0239262  ","3rd Qu.:0.0288933  ","Max.   :0.4264420  ",null],["Min.   :0.0006606  ","1st Qu.:0.0702120  ","Median :0.1001600  ","Mean   :0.1245312  ","3rd Qu.:0.1450440  ","Max.   :1.9595600  ",null],["Min.   : 0.00452  ","1st Qu.: 0.58407  ","Median : 0.91041  ","Mean   : 1.37753  ","3rd Qu.: 1.53524  ","Max.   :44.63360  ",null],["Min.   :  2.861  ","1st Qu.:  5.372  ","Median :  7.383  ","Mean   :  9.911  ","3rd Qu.: 11.451  ","Max.   :173.252  ",null],["Min.   :0.001174  ","1st Qu.:0.123960  ","Median :0.177284  ","Mean   :0.226270  ","3rd Qu.:0.259701  ","Max.   :4.035155  ",null],["Min.   :     0.0  ","1st Qu.:     3.0  ","Median :    16.0  ","Mean   :   291.1  ","3rd Qu.:   103.0  ","Max.   :114577.0  ",null],["Min.   :     0.0  ","1st Qu.:     3.0  ","Median :    16.0  ","Mean   :   291.1  ","3rd Qu.:   103.0  ","Max.   :114577.0  ",null],["Min.   :   0.256  ","1st Qu.:   4.624  ","Median :  10.888  ","Mean   :  26.466  ","3rd Qu.:  27.072  ","Max.   :2427.944  ",null],["Min.   :  0.0037  ","1st Qu.:  2.5256  ","Median :  6.4615  ","Mean   : 15.4715  ","3rd Qu.: 17.0854  ","Max.   :441.1374  ",null],["Min.   :0.0039  ","1st Qu.:0.4017  ","Median :0.5683  ","Mean   :0.6627  ","3rd Qu.:0.7932  ","Max.   :9.7635  ",null],["Min.   :0.000109  ","1st Qu.:0.055897  ","Median :0.112755  ","Mean   :0.204077  ","3rd Qu.:0.228158  ","Max.   :8.262894  ",null],["Min.   : 0.00452  ","1st Qu.: 0.58406  ","Median : 0.91040  ","Mean   : 1.37753  ","3rd Qu.: 1.53523  ","Max.   :44.63357  ",null],["Min.   :  2.861  ","1st Qu.:  5.372  ","Median :  7.383  ","Mean   :  9.911  ","3rd Qu.: 11.451  ","Max.   :173.252  ",null],["Min.   : 0.000073  ","1st Qu.: 0.047416  ","Median : 0.101692  ","Mean   : 0.207922  ","3rd Qu.: 0.225486  ","Max.   :10.891565  ",null],["Min.   :7.400e+01  ","1st Qu.:1.421e+04  ","Median :6.181e+04  ","Mean   :1.319e+06  ","3rd Qu.:4.073e+05  ","Max.   :2.629e+09  ",null],["Min.   :1.809e+03  ","1st Qu.:4.912e+07  ","Median :9.455e+08  ","Mean   :6.755e+12  ","3rd Qu.:3.370e+10  ","Max.   :4.338e+17  ",null],["Min.   :1.229e+04  ","1st Qu.:4.734e+10  ","Median :3.973e+12  ","Mean   :9.053e+19  ","3rd Qu.:6.675e+14  ","Max.   :2.908e+25  ",null],["Min.   :-6.100e+01  ","1st Qu.: 1.148e+08  ","Median : 1.177e+10  ","Mean   : 3.909e+16  ","3rd Qu.: 2.430e+12  ","Max.   : 8.380e+21  ",null],["Min.   : 0.2269  ","1st Qu.: 0.2594  ","Median : 0.3641  ","Mean   : 0.5311  ","3rd Qu.: 0.6554  ","Max.   :68.8837  ",null],["Min.   : 0.01699  ","1st Qu.: 0.02043  ","Median : 0.03105  ","Mean   : 0.07336  ","3rd Qu.: 0.07688  ","Max.   :21.69251  ",null],["Min.   :0.000366  ","1st Qu.:0.000486  ","Median :0.000701  ","Mean   :0.002645  ","3rd Qu.:0.001965  ","Max.   :6.417807  ",null],["Min.   :   0.000  ","1st Qu.:   0.000  ","Median :   0.004  ","Mean   :   0.177  ","3rd Qu.:   0.042  ","Max.   :3374.842  ",null],["Min.   :2.060e+02  ","1st Qu.:4.706e+05  ","Median :4.138e+06  ","Mean   :2.310e+09  ","3rd Qu.:8.185e+07  ","Max.   :1.633e+14  ",null],["Min.   :1.088e+04  ","1st Qu.:3.359e+10  ","Median :2.516e+12  ","Mean   :1.810e+20  ","3rd Qu.:7.258e+14  ","Max.   :3.583e+25  ",null],["Min.   :9.187e+03  ","1st Qu.:7.858e+10  ","Median :6.865e+12  ","Mean   :5.905e+22  ","3rd Qu.:3.435e+15  ","Max.   :2.665e+28  ",null],["Min.   :-2.200e+01  ","1st Qu.: 5.590e+07  ","Median : 6.551e+09  ","Mean   : 2.227e+16  ","3rd Qu.: 1.532e+12  ","Max.   : 4.527e+21  ",null],["Min.   :0.000e+00  ","1st Qu.:6.394e+06  ","Median :1.846e+09  ","Mean   :1.106e+16  ","3rd Qu.:6.960e+11  ","Max.   :1.957e+21  ",null],["Min.   :5.232e+03  ","1st Qu.:2.945e+09  ","Median :1.153e+11  ","Mean   :1.127e+18  ","3rd Qu.:1.715e+13  ","Max.   :3.743e+23  ",null],["Min.   :   0.060  ","1st Qu.:   0.088  ","Median :   0.210  ","Mean   :   0.769  ","3rd Qu.:   0.701  ","Max.   :8356.552  ",null],["Min.   :    0.00  ","1st Qu.:    0.00  ","Median :    0.01  ","Mean   :    0.37  ","3rd Qu.:    0.05  ","Max.   :48073.17  ",null],["Min.   :       0  ","1st Qu.:       0  ","Median :       0  ","Mean   :      82  ","3rd Qu.:       0  ","Max.   :69819221  ",null],["Min.   :   0.000  ","1st Qu.:   0.000  ","Median :   0.002  ","Mean   :   0.151  ","3rd Qu.:   0.027  ","Max.   :3367.004  ",null],["Min.   :   0.000  ","1st Qu.:   0.000  ","Median :   0.001  ","Mean   :   0.134  ","3rd Qu.:   0.014  ","Max.   :3361.778  ",null],["Min.   :     0.0  ","1st Qu.:     0.0  ","Median :     0.0  ","Mean   :     1.7  ","3rd Qu.:     0.3  ","Max.   :575558.1  ",null],["Min.   :    32  ","1st Qu.:   578  ","Median :  1361  ","Mean   :  3308  ","3rd Qu.:  3384  ","Max.   :303493  ",null],["Min.   :-153.74770  ","1st Qu.:  -0.61122  ","Median :   0.00013  ","Mean   :   0.03824  ","3rd Qu.:   0.64455  ","Max.   : 147.03809  ",null],["Min.   :0.0000309  ","1st Qu.:0.0816411  ","Median :0.1815433  ","Mean   :0.2560914  ","3rd Qu.:0.3958677  ","Max.   :0.9950984  ",null],["Min.   :0.0000494  ","1st Qu.:0.2104655  ","Median :0.3970670  ","Mean   :0.4316316  ","3rd Qu.:0.6381183  ","Max.   :1.0000000  ",null],["Min.   :0.009098  ","1st Qu.:0.381592  ","Median :0.595127  ","Mean   :0.565965  ","3rd Qu.:0.760363  ","Max.   :1.000000  ",null],["Min.   :  0.9297  ","1st Qu.:  3.4369  ","Median :  5.3158  ","Mean   :  7.5060  ","3rd Qu.:  9.5529  ","Max.   :202.7615  ",null],["Min.   : 0.528  ","1st Qu.: 2.254  ","Median : 3.189  ","Mean   : 4.080  ","3rd Qu.: 4.886  ","Max.   :34.516  ",null],["Min.   : 0.3033  ","1st Qu.: 1.7274  ","Median : 2.3752  ","Mean   : 2.6994  ","3rd Qu.: 3.2510  ","Max.   :20.3426  ",null],["Min.   :        3  ","1st Qu.:     7624  ","Median :    34098  ","Mean   :   682152  ","3rd Qu.:   199697  ","Max.   :567372596  ",null],["Min.   :2.000e+00  ","1st Qu.:1.402e+07  ","Median :2.824e+08  ","Mean   :1.035e+12  ","3rd Qu.:8.073e+09  ","Max.   :2.006e+16  ",null],["Min.   :1.000e+00  ","1st Qu.:7.148e+09  ","Median :6.350e+11  ","Mean   :1.152e+18  ","3rd Qu.:7.879e+13  ","Max.   :1.808e+23  ",null],["Min.   :-2.300e+01  ","1st Qu.: 2.743e+07  ","Median : 3.145e+09  ","Mean   : 2.832e+15  ","3rd Qu.: 6.405e+11  ","Max.   : 4.049e+20  ",null],["Min.   :  0.0329  ","1st Qu.:  0.3492  ","Median :  0.5615  ","Mean   :  0.7608  ","3rd Qu.:  0.9594  ","Max.   :412.3283  ",null],["Min.   :  0.00036  ","1st Qu.:  0.03507  ","Median :  0.07681  ","Mean   :  0.15403  ","3rd Qu.:  0.17312  ","Max.   :115.54759  ",null],["Min.   :0.00e+00  ","1st Qu.:1.01e-03  ","Median :2.70e-03  ","Mean   :8.42e-03  ","3rd Qu.:7.11e-03  ","Max.   :1.13e+02  ",null],["Min.   :    -0.04  ","1st Qu.:     0.00  ","Median :     0.01  ","Mean   :     1.10  ","3rd Qu.:     0.14  ","Max.   :197912.79  ",null],["Min.   :1.800e+01  ","1st Qu.:2.431e+05  ","Median :2.102e+06  ","Mean   :8.896e+08  ","3rd Qu.:3.965e+07  ","Max.   :1.282e+13  ",null],["Min.   :7.500e+01  ","1st Qu.:8.968e+09  ","Median :6.562e+11  ","Mean   :1.017e+19  ","3rd Qu.:1.656e+14  ","Max.   :1.509e+24  ",null],["Min.   :7.500e+01  ","1st Qu.:2.063e+10  ","Median :1.740e+12  ","Mean   :7.561e+20  ","3rd Qu.:8.401e+14  ","Max.   :1.643e+26  ",null],["Min.   :-6.000e+00  ","1st Qu.: 1.381e+07  ","Median : 1.881e+09  ","Mean   : 1.746e+15  ","3rd Qu.: 4.432e+11  ","Max.   : 2.342e+20  ",null],["Min.   :0.000e+00  ","1st Qu.:2.336e+06  ","Median :6.826e+08  ","Mean   :1.022e+15  ","3rd Qu.:2.376e+11  ","Max.   :1.203e+20  ",null],["Min.   :1.600e+01  ","1st Qu.:8.221e+08  ","Median :3.205e+10  ","Mean   :3.134e+16  ","3rd Qu.:4.148e+12  ","Max.   :5.255e+21  ",null],["Min.   :     0.00  ","1st Qu.:     0.17  ","Median :     0.49  ","Mean   :     2.54  ","3rd Qu.:     1.56  ","Max.   :298590.95  ",null],["Min.   :      0.0  ","1st Qu.:      0.0  ","Median :      0.0  ","Mean   :      4.6  ","3rd Qu.:      0.2  ","Max.   :2154055.6  ",null],["Min.   :0.000e+00  ","1st Qu.:0.000e+00  ","Median :0.000e+00  ","Mean   :1.220e+05  ","3rd Qu.:1.000e+00  ","Max.   :8.912e+10  ",null],["Min.   :    -0.02  ","1st Qu.:     0.00  ","Median :     0.01  ","Mean   :     1.07  ","3rd Qu.:     0.09  ","Max.   :264833.31  ",null],["Min.   :     0.00  ","1st Qu.:     0.00  ","Median :     0.00  ","Mean   :     1.05  ","3rd Qu.:     0.06  ","Max.   :309447.00  ",null],["Min.   :        0  ","1st Qu.:        0  ","Median :        0  ","Mean   :      218  ","3rd Qu.:        1  ","Max.   :123080053  ",null],["Min.   :    0.47  ","1st Qu.:  315.70  ","Median :  807.68  ","Mean   : 1933.94  ","3rd Qu.: 2135.68  ","Max.   :55142.18  ",null],["Min.   :-166.92892  ","1st Qu.:  -0.65125  ","Median :   0.00010  ","Mean   :   0.03832  ","3rd Qu.:   0.68048  ","Max.   : 167.53733  ",null],["Min.   :0.0000308  ","1st Qu.:0.0790912  ","Median :0.1821246  ","Mean   :0.2588727  ","3rd Qu.:0.4038948  ","Max.   :0.9967620  ",null],["Min.   :0.000049  ","1st Qu.:0.206639  ","Median :0.398185  ","Mean   :0.432130  ","3rd Qu.:0.642270  ","Max.   :1.000000  ",null],["Min.   :0.009933  ","1st Qu.:0.381986  ","Median :0.599260  ","Mean   :0.568147  ","3rd Qu.:0.765049  ","Max.   :1.000000  ",null],["Min.   :  0.9252  ","1st Qu.:  3.2311  ","Median :  5.0006  ","Mean   :  7.2324  ","3rd Qu.:  9.2722  ","Max.   :202.4823  ",null],["Min.   : 0.5272  ","1st Qu.: 2.1540  ","Median : 2.9887  ","Mean   : 3.8909  ","3rd Qu.: 4.6287  ","Max.   :33.1408  ",null],["Min.   : 0.3032  ","1st Qu.: 1.6675  ","Median : 2.2476  ","Mean   : 2.5643  ","3rd Qu.: 3.0394  ","Max.   :19.3785  ",null],["Min.   :  5.84  ","1st Qu.: 12.33  ","Median : 22.31  ","Mean   : 36.38  ","3rd Qu.: 47.29  ","Max.   :558.71  ",null],["Min.   :  2.764  ","1st Qu.: 11.747  ","Median : 18.025  ","Mean   : 22.892  ","3rd Qu.: 28.423  ","Max.   :269.172  ",null],["Min.   :  0.6818  ","1st Qu.:  6.6323  ","Median :  8.9127  ","Mean   : 16.1343  ","3rd Qu.: 19.8933  ","Max.   :366.9917  ",null],["Min.   :  3.662  ","1st Qu.:  8.694  ","Median : 14.601  ","Mean   : 25.323  ","3rd Qu.: 32.854  ","Max.   :446.136  ",null],["Min.   :  0.4922  ","1st Qu.:  4.7826  ","Median :  9.1547  ","Mean   : 13.5958  ","3rd Qu.: 17.7525  ","Max.   :208.1280  ",null],["Min.   :  3.978  ","1st Qu.: 11.077  ","Median : 20.870  ","Mean   : 30.788  ","3rd Qu.: 40.102  ","Max.   :455.101  ",null],["Min.   :  0.7943  ","1st Qu.:  9.2495  ","Median : 17.2526  ","Mean   : 27.7495  ","3rd Qu.: 37.1749  ","Max.   :476.2128  ",null],["Min.   :  2.476  ","1st Qu.:  8.868  ","Median : 15.664  ","Mean   : 21.633  ","3rd Qu.: 27.611  ","Max.   :297.276  ",null],["Min.   :  0.00243  ","1st Qu.:  4.30560  ","Median :  8.30746  ","Mean   : 13.21473  ","3rd Qu.: 17.13906  ","Max.   :299.01325  ",null],["Min.   :  1.588  ","1st Qu.: 15.509  ","Median : 24.622  ","Mean   : 33.282  ","3rd Qu.: 42.215  ","Max.   :420.809  ",null],["Min.   :  3.178  ","1st Qu.: 14.280  ","Median : 26.125  ","Mean   : 40.756  ","3rd Qu.: 53.948  ","Max.   :608.431  ",null],["Min.   :  0.0522  ","1st Qu.: 11.2541  ","Median : 18.4752  ","Mean   : 24.2844  ","3rd Qu.: 31.0972  ","Max.   :326.5259  ",null],["Min.   :  2.17  ","1st Qu.: 12.38  ","Median : 23.10  ","Mean   : 36.64  ","3rd Qu.: 48.83  ","Max.   :562.20  ",null],["Min.   :  0.7449  ","1st Qu.:  5.4942  ","Median :  9.9844  ","Mean   : 16.6032  ","3rd Qu.: 21.9382  ","Max.   :315.5733  ",null],["Min.   :  2.692  ","1st Qu.:  8.492  ","Median : 17.013  ","Mean   : 25.411  ","3rd Qu.: 33.349  ","Max.   :407.494  ",null],["Min.   :  2.174  ","1st Qu.: 14.591  ","Median : 26.231  ","Mean   : 38.001  ","3rd Qu.: 49.903  ","Max.   :534.474  ",null],["Min.   :0.673  ","1st Qu.:1.296  ","Median :1.479  ","Mean   :1.540  ","3rd Qu.:1.718  ","Max.   :5.053  ",null],["Min.   :  0.8848  ","1st Qu.: 11.6501  ","Median : 22.2761  ","Mean   : 32.4719  ","3rd Qu.: 43.2435  ","Max.   :465.6008  ",null],["Min.   :  4.534  ","1st Qu.: 10.606  ","Median : 19.246  ","Mean   : 32.272  ","3rd Qu.: 42.101  ","Max.   :530.322  ",null],["Min.   :  0.000  ","1st Qu.:  6.055  ","Median : 11.989  ","Mean   : 17.711  ","3rd Qu.: 23.916  ","Max.   :313.093  ",null],["Min.   :  2.597  ","1st Qu.:  9.565  ","Median : 16.809  ","Mean   : 28.317  ","3rd Qu.: 36.569  ","Max.   :211.119  ",null],["Min.   :  0.334  ","1st Qu.:  8.681  ","Median : 13.886  ","Mean   : 17.533  ","3rd Qu.: 22.580  ","Max.   :114.735  ",null],["Min.   :  0.6197  ","1st Qu.:  6.2461  ","Median :  8.0175  ","Mean   : 14.3717  ","3rd Qu.: 17.6041  ","Max.   :126.9188  ",null],["Min.   :  1.906  ","1st Qu.:  7.530  ","Median : 11.831  ","Mean   : 21.007  ","3rd Qu.: 27.021  ","Max.   :159.919  ",null],["Min.   : 0.4221  ","1st Qu.: 4.1533  ","Median : 6.9654  ","Mean   :10.7682  ","3rd Qu.:13.7894  ","Max.   :88.5386  ",null],["Min.   :  2.151  ","1st Qu.:  8.331  ","Median : 15.600  ","Mean   : 23.710  ","3rd Qu.: 30.959  ","Max.   :182.183  ",null],["Min.   :  0.1964  ","1st Qu.:  5.9587  ","Median : 14.2866  ","Mean   : 22.3958  ","3rd Qu.: 31.0394  ","Max.   :196.0675  ",null],["Min.   :  1.001  ","1st Qu.:  6.467  ","Median : 11.295  ","Mean   : 16.230  ","3rd Qu.: 20.766  ","Max.   :120.864  ",null],["Min.   :  0.00053  ","1st Qu.:  2.90179  ","Median :  6.41665  ","Mean   : 11.48210  ","3rd Qu.: 15.82929  ","Max.   :122.49338  ",null],["Min.   :  0.3559  ","1st Qu.: 12.0308  ","Median : 19.4040  ","Mean   : 25.6229  ","3rd Qu.: 32.4154  ","Max.   :176.3805  ",null],["Min.   :  0.3606  ","1st Qu.:  9.4249  ","Median : 20.2085  ","Mean   : 31.2807  ","3rd Qu.: 42.0616  ","Max.   :282.8048  ",null],["Min.   :  0.02937  ","1st Qu.:  9.07551  ","Median : 15.12503  ","Mean   : 19.59048  ","3rd Qu.: 25.80373  ","Max.   :135.27674  ",null],["Min.   :  0.2664  ","1st Qu.:  8.1433  ","Median : 18.3868  ","Mean   : 28.5944  ","3rd Qu.: 38.8146  ","Max.   :263.8942  ",null],["Min.   :  0.6353  ","1st Qu.:  5.2433  ","Median :  8.4113  ","Mean   : 14.1454  ","3rd Qu.: 18.2614  ","Max.   :118.2292  ",null],["Min.   :  1.683  ","1st Qu.:  6.984  ","Median : 13.156  ","Mean   : 20.244  ","3rd Qu.: 26.615  ","Max.   :167.080  ",null],["Min.   :  0.2699  ","1st Qu.: 11.0095  ","Median : 20.6910  ","Mean   : 29.2589  ","3rd Qu.: 38.5569  ","Max.   :236.6704  ",null],["Min.   :0.6087  ","1st Qu.:1.2838  ","Median :1.4695  ","Mean   :1.5314  ","3rd Qu.:1.7144  ","Max.   :5.0508  ",null],["Min.   :  0.1952  ","1st Qu.:  9.4088  ","Median : 18.2436  ","Mean   : 25.6412  ","3rd Qu.: 34.2142  ","Max.   :204.5134  ",null],["Min.   :  2.256  ","1st Qu.:  8.640  ","Median : 14.947  ","Mean   : 25.710  ","3rd Qu.: 33.289  ","Max.   :195.119  ",null],["Min.   :  0.00512  ","1st Qu.:  4.92828  ","Median : 10.78334  ","Mean   : 15.20029  ","3rd Qu.: 21.08573  ","Max.   :122.57626  ",null],["Min.   :0.4801  ","1st Qu.:1.7996  ","Median :2.0510  ","Mean   :2.1389  ","3rd Qu.:2.4500  ","Max.   :8.9997  ",null],["Min.   :-1.942e-07  ","1st Qu.:-4.640e-11  ","Median : 8.000e-13  ","Mean   : 4.570e-11  ","3rd Qu.: 5.040e-11  ","Max.   : 3.621e-07  ",null],["Min.   :0.00125  ","1st Qu.:0.09093  ","Median :0.12314  ","Mean   :0.12955  ","3rd Qu.:0.15969  ","Max.   :0.94189  ",null],["Min.   :0.0000016  ","1st Qu.:0.0082679  ","Median :0.0151631  ","Mean   :0.0196832  ","3rd Qu.:0.0255021  ","Max.   :0.8871492  ",null],["Min.   :-10.82110  ","1st Qu.: -0.85198  ","Median : -0.66846  ","Mean   : -0.70352  ","3rd Qu.: -0.50348  ","Max.   : -0.01075  ",null],["Min.   : 0.00718  ","1st Qu.: 1.15784  ","Median : 1.87047  ","Mean   : 2.63039  ","3rd Qu.: 3.10925  ","Max.   :45.26153  ",null]],"container":"<table class=\"table table-striped table-hover\">\n  <thead>\n    <tr>\n      <th>local_res_atom_non_h_count<\/th>\n      <th>local_res_atom_non_h_electron_sum<\/th>\n      <th>dict_atom_non_h_count<\/th>\n      <th>dict_atom_non_h_electron_sum<\/th>\n      <th> local_volume<\/th>\n      <th>local_electrons<\/th>\n      <th>  local_mean<\/th>\n      <th>  local_std<\/th>\n      <th>  local_max<\/th>\n      <th>local_max_over_std<\/th>\n      <th>local_skewness<\/th>\n      <th>shape_segments_count<\/th>\n      <th>density_segments_count<\/th>\n      <th>    volume<\/th>\n      <th>  electrons<\/th>\n      <th>     mean<\/th>\n      <th>     std<\/th>\n      <th>     max<\/th>\n      <th> max_over_std<\/th>\n      <th>   skewness<\/th>\n      <th>   shape_O3<\/th>\n      <th>   shape_O4<\/th>\n      <th>   shape_O5<\/th>\n      <th>   shape_FL<\/th>\n      <th>shape_O3_norm<\/th>\n      <th>shape_O4_norm<\/th>\n      <th>shape_O5_norm<\/th>\n      <th>shape_FL_norm<\/th>\n      <th>   shape_I1<\/th>\n      <th>   shape_I2<\/th>\n      <th>   shape_I3<\/th>\n      <th>   shape_I4<\/th>\n      <th>   shape_I5<\/th>\n      <th>   shape_I6<\/th>\n      <th>shape_I1_norm<\/th>\n      <th>shape_I2_norm<\/th>\n      <th>shape_I3_norm<\/th>\n      <th>shape_I4_norm<\/th>\n      <th>shape_I5_norm<\/th>\n      <th>shape_I6_norm<\/th>\n      <th>  shape_M000<\/th>\n      <th>   shape_CI<\/th>\n      <th> shape_E3_E1<\/th>\n      <th> shape_E2_E1<\/th>\n      <th> shape_E3_E2<\/th>\n      <th>shape_sqrt_E1<\/th>\n      <th>shape_sqrt_E2<\/th>\n      <th>shape_sqrt_E3<\/th>\n      <th>  density_O3<\/th>\n      <th>  density_O4<\/th>\n      <th>  density_O5<\/th>\n      <th>  density_FL<\/th>\n      <th>density_O3_norm<\/th>\n      <th>density_O4_norm<\/th>\n      <th>density_O5_norm<\/th>\n      <th>density_FL_norm<\/th>\n      <th>  density_I1<\/th>\n      <th>  density_I2<\/th>\n      <th>  density_I3<\/th>\n      <th>  density_I4<\/th>\n      <th>  density_I5<\/th>\n      <th>  density_I6<\/th>\n      <th>density_I1_norm<\/th>\n      <th>density_I2_norm<\/th>\n      <th>density_I3_norm<\/th>\n      <th>density_I4_norm<\/th>\n      <th>density_I5_norm<\/th>\n      <th>density_I6_norm<\/th>\n      <th> density_M000<\/th>\n      <th>  density_CI<\/th>\n      <th>density_E3_E1<\/th>\n      <th>density_E2_E1<\/th>\n      <th>density_E3_E2<\/th>\n      <th>density_sqrt_E1<\/th>\n      <th>density_sqrt_E2<\/th>\n      <th>density_sqrt_E3<\/th>\n      <th> shape_Z_7_3<\/th>\n      <th> shape_Z_0_0<\/th>\n      <th> shape_Z_7_0<\/th>\n      <th> shape_Z_7_1<\/th>\n      <th> shape_Z_3_0<\/th>\n      <th> shape_Z_5_2<\/th>\n      <th> shape_Z_6_1<\/th>\n      <th> shape_Z_3_1<\/th>\n      <th> shape_Z_6_0<\/th>\n      <th> shape_Z_2_1<\/th>\n      <th> shape_Z_6_3<\/th>\n      <th> shape_Z_2_0<\/th>\n      <th> shape_Z_6_2<\/th>\n      <th> shape_Z_5_0<\/th>\n      <th> shape_Z_5_1<\/th>\n      <th> shape_Z_4_2<\/th>\n      <th> shape_Z_1_0<\/th>\n      <th> shape_Z_4_1<\/th>\n      <th> shape_Z_7_2<\/th>\n      <th> shape_Z_4_0<\/th>\n      <th>density_Z_7_3<\/th>\n      <th>density_Z_0_0<\/th>\n      <th>density_Z_7_0<\/th>\n      <th>density_Z_7_1<\/th>\n      <th>density_Z_3_0<\/th>\n      <th>density_Z_5_2<\/th>\n      <th>density_Z_6_1<\/th>\n      <th>density_Z_3_1<\/th>\n      <th>density_Z_6_0<\/th>\n      <th>density_Z_2_1<\/th>\n      <th>density_Z_6_3<\/th>\n      <th>density_Z_2_0<\/th>\n      <th>density_Z_6_2<\/th>\n      <th>density_Z_5_0<\/th>\n      <th>density_Z_5_1<\/th>\n      <th>density_Z_4_2<\/th>\n      <th>density_Z_1_0<\/th>\n      <th>density_Z_4_1<\/th>\n      <th>density_Z_7_2<\/th>\n      <th>density_Z_4_0<\/th>\n      <th>  resolution<\/th>\n      <th>  FoFc_mean<\/th>\n      <th>   FoFc_std<\/th>\n      <th>FoFc_square_std<\/th>\n      <th>   FoFc_min<\/th>\n      <th>   FoFc_max<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"dom":"Bfrtip","buttons":["copy","csv","excel","pdf","print"],"scrollX":true,"order":[],"autoWidth":false,"orderClasses":false,"rowCallback":"function(row, data) {\n}"}},"evals":["options.rowCallback"],"jsHooks":[]}</script><!--/html_preserve-->

# 7. Kod ograniczaj�cy liczb� klas (res_name) do 50 najpopularniejszych warto�ci.

```r
res_name50 <- data %>% group_by(res_name) %>% summarize(n = n()) %>% arrange(desc(n)) %>% head(50)
data <- data %>% filter(res_name %in% res_name50$res_name)
rm(res_name50)
```

# 8. Sekcj� sprawdzaj�c� korelacje mi�dzy zmiennymi
Poni�ej znajduje si� wykres korelacji mi�dzy zmiennymi. Zauwa�y� mo�na, �e:
* Du�a cz�� zmiennych jest w silnej korelacji z innymi zmiennymi
* Zmienne density_Z* i shape_z* s� w bardzo silnej korelacji (du�e czerwone prostok�ty)
* Zmienne local* koreluj� z zmiennymi z part (by�y obliczane na ich podstawie)

```r
melted <- data %>% select(-res_name, -part) %>% cor() %>% melt
breaks <- sort(colnames(data))[seq(1, ncol(data), by = 6)]

(ggplot(data = melted, aes(x=X1, y=X2, fill=value)) + 
    theme(
        axis.title.x = element_blank(),
        axis.title.y = element_blank(),
        axis.text.x = element_text(angle=45)
      ) + 
    scale_x_discrete(breaks = breaks) + 
    scale_y_discrete(breaks = breaks) + 
    geom_tile() +
    scale_fill_gradient2(
        low = "blue", high = "red", mid = "white", 
        midpoint = 0, limit = c(-1,1))
      ) %>%
   ggplotly()
```

<!--html_preserve--><div id="htmlwidget-e3b1a3fea7b58b984a25" style="width:672px;height:480px;" class="plotly html-widget"></div>

```r
rm(melted, breaks)
```

# 9. Okre�lenie ile przyk�ad�w ma ka�da z klas.

```r
data %>% select(res_name) %>% group_by(res_name) %>% summarize(n = n()) %>% arrange(desc(n)) %>% prettyTable()
```

<!--html_preserve--><div id="htmlwidget-f6e6b0cb067ac237513b" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-f6e6b0cb067ac237513b">{"x":{"style":"bootstrap","filter":"none","data":[["SO4","GOL","EDO","NAG","CL","CA","ZN","MG","PO4","HEM","NA","ACT","DMS","IOD","PEG","K","FAD","NAD","MN","CLA","ADP","NAP","MLY","CD","MPD","FMT","MES","PG4","MAN","CU","ATP","BR","COA","NDP","FMN","1PE","EPE","HEC","PGE","SF4","NI","TRS","FE","PLP","ACY","NO3","GDP","SAH","FE2","SEP"],[168078,118234,89471,74339,69092,62670,59057,43292,32956,32853,28460,23732,19693,18875,14214,13992,13517,13216,12565,11614,11255,10363,10160,9697,9404,8527,7948,7935,7670,7041,6797,6357,6323,6202,6158,6130,5660,5658,5469,4936,4886,4882,4793,4747,4735,4728,4717,4708,4667,4387]],"container":"<table class=\"table table-striped table-hover\">\n  <thead>\n    <tr>\n      <th>res_name<\/th>\n      <th>n<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"dom":"Bfrtip","buttons":["copy","csv","excel","pdf","print"],"scrollX":true,"columnDefs":[{"className":"dt-right","targets":1}],"order":[],"autoWidth":false,"orderClasses":false,"rowCallback":"function(row, data) {\n}"}},"evals":["options.rowCallback"],"jsHooks":[]}</script><!--/html_preserve-->

# 10. Wykresy rozk�ad�w liczby atom�w i elektron�w.

```r
(ggplot(data, aes(local_res_atom_non_h_count, fill = "red")) + geom_bar() + theme_bw()) %>% ggplotly()
```

<!--html_preserve--><div id="htmlwidget-246a695e27233b7744db" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-246a695e27233b7744db">{"x":{"data":[{"orientation":"v","width":[0.9,0.9,0.9,0.9,0.9,0.9,0.9,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.899999999999999,0.900000000000002,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006,0.900000000000006],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,64,65],"y":[345483,21,8795,143109,201269,118569,14879,19439,1379,10689,16255,9491,6613,72609,11048,4001,45,71,32,6,14,36,106,15,275,4749,11867,4702,21,141,13355,106,30,6,214,93,6,72,310,79,132,111,38391,12661,305,404,178,21034,95,428,142,139,13295,166,542,51,37,64,89,424,64,103,11,7994],"text":["count: 345483<br />local_res_atom_non_h_count:  1<br />fill: red","count:     21<br />local_res_atom_non_h_count:  2<br />fill: red","count:   8795<br />local_res_atom_non_h_count:  3<br />fill: red","count: 143109<br />local_res_atom_non_h_count:  4<br />fill: red","count: 201269<br />local_res_atom_non_h_count:  5<br />fill: red","count: 118569<br />local_res_atom_non_h_count:  6<br />fill: red","count:  14879<br />local_res_atom_non_h_count:  7<br />fill: red","count:  19439<br />local_res_atom_non_h_count:  8<br />fill: red","count:   1379<br />local_res_atom_non_h_count:  9<br />fill: red","count:  10689<br />local_res_atom_non_h_count: 10<br />fill: red","count:  16255<br />local_res_atom_non_h_count: 11<br />fill: red","count:   9491<br />local_res_atom_non_h_count: 12<br />fill: red","count:   6613<br />local_res_atom_non_h_count: 13<br />fill: red","count:  72609<br />local_res_atom_non_h_count: 14<br />fill: red","count:  11048<br />local_res_atom_non_h_count: 15<br />fill: red","count:   4001<br />local_res_atom_non_h_count: 16<br />fill: red","count:     45<br />local_res_atom_non_h_count: 17<br />fill: red","count:     71<br />local_res_atom_non_h_count: 18<br />fill: red","count:     32<br />local_res_atom_non_h_count: 19<br />fill: red","count:      6<br />local_res_atom_non_h_count: 20<br />fill: red","count:     14<br />local_res_atom_non_h_count: 21<br />fill: red","count:     36<br />local_res_atom_non_h_count: 22<br />fill: red","count:    106<br />local_res_atom_non_h_count: 23<br />fill: red","count:     15<br />local_res_atom_non_h_count: 24<br />fill: red","count:    275<br />local_res_atom_non_h_count: 25<br />fill: red","count:   4749<br />local_res_atom_non_h_count: 26<br />fill: red","count:  11867<br />local_res_atom_non_h_count: 27<br />fill: red","count:   4702<br />local_res_atom_non_h_count: 28<br />fill: red","count:     21<br />local_res_atom_non_h_count: 29<br />fill: red","count:    141<br />local_res_atom_non_h_count: 30<br />fill: red","count:  13355<br />local_res_atom_non_h_count: 31<br />fill: red","count:    106<br />local_res_atom_non_h_count: 32<br />fill: red","count:     30<br />local_res_atom_non_h_count: 33<br />fill: red","count:      6<br />local_res_atom_non_h_count: 34<br />fill: red","count:    214<br />local_res_atom_non_h_count: 35<br />fill: red","count:     93<br />local_res_atom_non_h_count: 36<br />fill: red","count:      6<br />local_res_atom_non_h_count: 37<br />fill: red","count:     72<br />local_res_atom_non_h_count: 38<br />fill: red","count:    310<br />local_res_atom_non_h_count: 39<br />fill: red","count:     79<br />local_res_atom_non_h_count: 40<br />fill: red","count:    132<br />local_res_atom_non_h_count: 41<br />fill: red","count:    111<br />local_res_atom_non_h_count: 42<br />fill: red","count:  38391<br />local_res_atom_non_h_count: 43<br />fill: red","count:  12661<br />local_res_atom_non_h_count: 44<br />fill: red","count:    305<br />local_res_atom_non_h_count: 45<br />fill: red","count:    404<br />local_res_atom_non_h_count: 46<br />fill: red","count:    178<br />local_res_atom_non_h_count: 47<br />fill: red","count:  21034<br />local_res_atom_non_h_count: 48<br />fill: red","count:     95<br />local_res_atom_non_h_count: 49<br />fill: red","count:    428<br />local_res_atom_non_h_count: 50<br />fill: red","count:    142<br />local_res_atom_non_h_count: 51<br />fill: red","count:    139<br />local_res_atom_non_h_count: 52<br />fill: red","count:  13295<br />local_res_atom_non_h_count: 53<br />fill: red","count:    166<br />local_res_atom_non_h_count: 54<br />fill: red","count:    542<br />local_res_atom_non_h_count: 55<br />fill: red","count:     51<br />local_res_atom_non_h_count: 56<br />fill: red","count:     37<br />local_res_atom_non_h_count: 57<br />fill: red","count:     64<br />local_res_atom_non_h_count: 58<br />fill: red","count:     89<br />local_res_atom_non_h_count: 59<br />fill: red","count:    424<br />local_res_atom_non_h_count: 60<br />fill: red","count:     64<br />local_res_atom_non_h_count: 61<br />fill: red","count:    103<br />local_res_atom_non_h_count: 62<br />fill: red","count:     11<br />local_res_atom_non_h_count: 64<br />fill: red","count:   7994<br />local_res_atom_non_h_count: 65<br />fill: red"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(248,118,109,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"red","legendgroup":"red","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":26.2283105022831,"r":7.30593607305936,"b":40.1826484018265,"l":54.7945205479452},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-2.695,68.695],"tickmode":"array","ticktext":["0","20","40","60"],"tickvals":[0,20,40,60],"categoryorder":"array","categoryarray":["0","20","40","60"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":"local_res_atom_non_h_count","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-17274.15,362757.15],"tickmode":"array","ticktext":["0e+00","1e+05","2e+05","3e+05"],"tickvals":[0,100000,200000,300000],"categoryorder":"array","categoryarray":["0e+00","1e+05","2e+05","3e+05"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":"count","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(51,51,51,1)","width":0.66417600664176,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895},"y":0.93503937007874},"annotations":[{"text":"fill","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"cloud":false},"source":"A","attrs":{"12f067f45f75":{"x":{},"fill":{},"type":"bar"}},"cur_data":"12f067f45f75","visdat":{"12f067f45f75":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->

```r
(ggplot(data, aes(local_res_atom_non_h_electron_sum, fill = "red")) + geom_bar(width = 3) + theme_bw()) %>% ggplotly()
```

<!--html_preserve--><div id="htmlwidget-0b71cc1f8f3420c0e55f" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-0b71cc1f8f3420c0e55f">{"x":{"data":[{"orientation":"v","width":[3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3],"base":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"x":[6,8,11,12,14,15,16,17,19,20,22,24,25,26,27,28,29,30,31,32,33,34,35,36,38,39,40,41,42,44,45,46,47,48,51,52,53,54,55,56,58,59,60,61,62,64,65,66,67,68,70,71,72,73,74,75,76,78,79,80,81,83,84,86,87,88,89,90,91,92,94,95,96,100,102,103,104,108,110,112,113,117,118,120,122,125,127,128,135,140,142,144,147,151,152,158,160,164,166,168,171,172,174,178,179,182,185,186,191,196,197,198,205,206,209,211,213,217,222,228,230,234,236,238,244,245,249,250,256,259,262,264,266,268,274,278,282,284,288,290,296,298,299,302,305,308,309,310,314,315,317,320,326,332,335,336,338,341,344,348,349,350,356,362,364,367,368,372,374,375,380,386,392,402,404,410],"y":[6,6,28460,43292,3,6,24,69092,13992,62805,8550,15,12565,9476,21,123354,7041,59060,4772,104,117,265,6357,19693,14,165,486,32,118022,5,97,160,32803,192310,45,9401,18875,259,4826,11,946,26,372,9,6,72,18,228,6,6170,8831,39,117,4,370,5,6757,80,3,4680,12,24,907,99,112,6470,12,151,7872,21,93,72413,15,68,46,1763,36,3309,9190,27,21,6,689,9,15,58,11,65,3,15,12,6,3,3,6,9,247,6,82,4921,9,9,22,60,17,4674,15,9,6,9,12,48,11807,15,41,3,4699,6114,3,83,6,3,97,3,7247,6,12,106,53,208,88,3,19,42,19,72,27,13,3,38616,423,246,22,149,76,31,46,23,95,51,12645,428,142,100,3,18,4,3,166,5,12,542,15351,37,5712,36,67,4,85,13291,424,64,103,6,5,7994],"text":["count:      6<br />local_res_atom_non_h_electron_sum:   6<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum:   8<br />fill: red","count:  28460<br />local_res_atom_non_h_electron_sum:  11<br />fill: red","count:  43292<br />local_res_atom_non_h_electron_sum:  12<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum:  14<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum:  15<br />fill: red","count:     24<br />local_res_atom_non_h_electron_sum:  16<br />fill: red","count:  69092<br />local_res_atom_non_h_electron_sum:  17<br />fill: red","count:  13992<br />local_res_atom_non_h_electron_sum:  19<br />fill: red","count:  62805<br />local_res_atom_non_h_electron_sum:  20<br />fill: red","count:   8550<br />local_res_atom_non_h_electron_sum:  22<br />fill: red","count:     15<br />local_res_atom_non_h_electron_sum:  24<br />fill: red","count:  12565<br />local_res_atom_non_h_electron_sum:  25<br />fill: red","count:   9476<br />local_res_atom_non_h_electron_sum:  26<br />fill: red","count:     21<br />local_res_atom_non_h_electron_sum:  27<br />fill: red","count: 123354<br />local_res_atom_non_h_electron_sum:  28<br />fill: red","count:   7041<br />local_res_atom_non_h_electron_sum:  29<br />fill: red","count:  59060<br />local_res_atom_non_h_electron_sum:  30<br />fill: red","count:   4772<br />local_res_atom_non_h_electron_sum:  31<br />fill: red","count:    104<br />local_res_atom_non_h_electron_sum:  32<br />fill: red","count:    117<br />local_res_atom_non_h_electron_sum:  33<br />fill: red","count:    265<br />local_res_atom_non_h_electron_sum:  34<br />fill: red","count:   6357<br />local_res_atom_non_h_electron_sum:  35<br />fill: red","count:  19693<br />local_res_atom_non_h_electron_sum:  36<br />fill: red","count:     14<br />local_res_atom_non_h_electron_sum:  38<br />fill: red","count:    165<br />local_res_atom_non_h_electron_sum:  39<br />fill: red","count:    486<br />local_res_atom_non_h_electron_sum:  40<br />fill: red","count:     32<br />local_res_atom_non_h_electron_sum:  41<br />fill: red","count: 118022<br />local_res_atom_non_h_electron_sum:  42<br />fill: red","count:      5<br />local_res_atom_non_h_electron_sum:  44<br />fill: red","count:     97<br />local_res_atom_non_h_electron_sum:  45<br />fill: red","count:    160<br />local_res_atom_non_h_electron_sum:  46<br />fill: red","count:  32803<br />local_res_atom_non_h_electron_sum:  47<br />fill: red","count: 192310<br />local_res_atom_non_h_electron_sum:  48<br />fill: red","count:     45<br />local_res_atom_non_h_electron_sum:  51<br />fill: red","count:   9401<br />local_res_atom_non_h_electron_sum:  52<br />fill: red","count:  18875<br />local_res_atom_non_h_electron_sum:  53<br />fill: red","count:    259<br />local_res_atom_non_h_electron_sum:  54<br />fill: red","count:   4826<br />local_res_atom_non_h_electron_sum:  55<br />fill: red","count:     11<br />local_res_atom_non_h_electron_sum:  56<br />fill: red","count:    946<br />local_res_atom_non_h_electron_sum:  58<br />fill: red","count:     26<br />local_res_atom_non_h_electron_sum:  59<br />fill: red","count:    372<br />local_res_atom_non_h_electron_sum:  60<br />fill: red","count:      9<br />local_res_atom_non_h_electron_sum:  61<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum:  62<br />fill: red","count:     72<br />local_res_atom_non_h_electron_sum:  64<br />fill: red","count:     18<br />local_res_atom_non_h_electron_sum:  65<br />fill: red","count:    228<br />local_res_atom_non_h_electron_sum:  66<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum:  67<br />fill: red","count:   6170<br />local_res_atom_non_h_electron_sum:  68<br />fill: red","count:   8831<br />local_res_atom_non_h_electron_sum:  70<br />fill: red","count:     39<br />local_res_atom_non_h_electron_sum:  71<br />fill: red","count:    117<br />local_res_atom_non_h_electron_sum:  72<br />fill: red","count:      4<br />local_res_atom_non_h_electron_sum:  73<br />fill: red","count:    370<br />local_res_atom_non_h_electron_sum:  74<br />fill: red","count:      5<br />local_res_atom_non_h_electron_sum:  75<br />fill: red","count:   6757<br />local_res_atom_non_h_electron_sum:  76<br />fill: red","count:     80<br />local_res_atom_non_h_electron_sum:  78<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum:  79<br />fill: red","count:   4680<br />local_res_atom_non_h_electron_sum:  80<br />fill: red","count:     12<br />local_res_atom_non_h_electron_sum:  81<br />fill: red","count:     24<br />local_res_atom_non_h_electron_sum:  83<br />fill: red","count:    907<br />local_res_atom_non_h_electron_sum:  84<br />fill: red","count:     99<br />local_res_atom_non_h_electron_sum:  86<br />fill: red","count:    112<br />local_res_atom_non_h_electron_sum:  87<br />fill: red","count:   6470<br />local_res_atom_non_h_electron_sum:  88<br />fill: red","count:     12<br />local_res_atom_non_h_electron_sum:  89<br />fill: red","count:    151<br />local_res_atom_non_h_electron_sum:  90<br />fill: red","count:   7872<br />local_res_atom_non_h_electron_sum:  91<br />fill: red","count:     21<br />local_res_atom_non_h_electron_sum:  92<br />fill: red","count:     93<br />local_res_atom_non_h_electron_sum:  94<br />fill: red","count:  72413<br />local_res_atom_non_h_electron_sum:  95<br />fill: red","count:     15<br />local_res_atom_non_h_electron_sum:  96<br />fill: red","count:     68<br />local_res_atom_non_h_electron_sum: 100<br />fill: red","count:     46<br />local_res_atom_non_h_electron_sum: 102<br />fill: red","count:   1763<br />local_res_atom_non_h_electron_sum: 103<br />fill: red","count:     36<br />local_res_atom_non_h_electron_sum: 104<br />fill: red","count:   3309<br />local_res_atom_non_h_electron_sum: 108<br />fill: red","count:   9190<br />local_res_atom_non_h_electron_sum: 110<br />fill: red","count:     27<br />local_res_atom_non_h_electron_sum: 112<br />fill: red","count:     21<br />local_res_atom_non_h_electron_sum: 113<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum: 117<br />fill: red","count:    689<br />local_res_atom_non_h_electron_sum: 118<br />fill: red","count:      9<br />local_res_atom_non_h_electron_sum: 120<br />fill: red","count:     15<br />local_res_atom_non_h_electron_sum: 122<br />fill: red","count:     58<br />local_res_atom_non_h_electron_sum: 125<br />fill: red","count:     11<br />local_res_atom_non_h_electron_sum: 127<br />fill: red","count:     65<br />local_res_atom_non_h_electron_sum: 128<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum: 135<br />fill: red","count:     15<br />local_res_atom_non_h_electron_sum: 140<br />fill: red","count:     12<br />local_res_atom_non_h_electron_sum: 142<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum: 144<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum: 147<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum: 151<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum: 152<br />fill: red","count:      9<br />local_res_atom_non_h_electron_sum: 158<br />fill: red","count:    247<br />local_res_atom_non_h_electron_sum: 160<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum: 164<br />fill: red","count:     82<br />local_res_atom_non_h_electron_sum: 166<br />fill: red","count:   4921<br />local_res_atom_non_h_electron_sum: 168<br />fill: red","count:      9<br />local_res_atom_non_h_electron_sum: 171<br />fill: red","count:      9<br />local_res_atom_non_h_electron_sum: 172<br />fill: red","count:     22<br />local_res_atom_non_h_electron_sum: 174<br />fill: red","count:     60<br />local_res_atom_non_h_electron_sum: 178<br />fill: red","count:     17<br />local_res_atom_non_h_electron_sum: 179<br />fill: red","count:   4674<br />local_res_atom_non_h_electron_sum: 182<br />fill: red","count:     15<br />local_res_atom_non_h_electron_sum: 185<br />fill: red","count:      9<br />local_res_atom_non_h_electron_sum: 186<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum: 191<br />fill: red","count:      9<br />local_res_atom_non_h_electron_sum: 196<br />fill: red","count:     12<br />local_res_atom_non_h_electron_sum: 197<br />fill: red","count:     48<br />local_res_atom_non_h_electron_sum: 198<br />fill: red","count:  11807<br />local_res_atom_non_h_electron_sum: 205<br />fill: red","count:     15<br />local_res_atom_non_h_electron_sum: 206<br />fill: red","count:     41<br />local_res_atom_non_h_electron_sum: 209<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum: 211<br />fill: red","count:   4699<br />local_res_atom_non_h_electron_sum: 213<br />fill: red","count:   6114<br />local_res_atom_non_h_electron_sum: 217<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum: 222<br />fill: red","count:     83<br />local_res_atom_non_h_electron_sum: 228<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum: 230<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum: 234<br />fill: red","count:     97<br />local_res_atom_non_h_electron_sum: 236<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum: 238<br />fill: red","count:   7247<br />local_res_atom_non_h_electron_sum: 244<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum: 245<br />fill: red","count:     12<br />local_res_atom_non_h_electron_sum: 249<br />fill: red","count:    106<br />local_res_atom_non_h_electron_sum: 250<br />fill: red","count:     53<br />local_res_atom_non_h_electron_sum: 256<br />fill: red","count:    208<br />local_res_atom_non_h_electron_sum: 259<br />fill: red","count:     88<br />local_res_atom_non_h_electron_sum: 262<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum: 264<br />fill: red","count:     19<br />local_res_atom_non_h_electron_sum: 266<br />fill: red","count:     42<br />local_res_atom_non_h_electron_sum: 268<br />fill: red","count:     19<br />local_res_atom_non_h_electron_sum: 274<br />fill: red","count:     72<br />local_res_atom_non_h_electron_sum: 278<br />fill: red","count:     27<br />local_res_atom_non_h_electron_sum: 282<br />fill: red","count:     13<br />local_res_atom_non_h_electron_sum: 284<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum: 288<br />fill: red","count:  38616<br />local_res_atom_non_h_electron_sum: 290<br />fill: red","count:    423<br />local_res_atom_non_h_electron_sum: 296<br />fill: red","count:    246<br />local_res_atom_non_h_electron_sum: 298<br />fill: red","count:     22<br />local_res_atom_non_h_electron_sum: 299<br />fill: red","count:    149<br />local_res_atom_non_h_electron_sum: 302<br />fill: red","count:     76<br />local_res_atom_non_h_electron_sum: 305<br />fill: red","count:     31<br />local_res_atom_non_h_electron_sum: 308<br />fill: red","count:     46<br />local_res_atom_non_h_electron_sum: 309<br />fill: red","count:     23<br />local_res_atom_non_h_electron_sum: 310<br />fill: red","count:     95<br />local_res_atom_non_h_electron_sum: 314<br />fill: red","count:     51<br />local_res_atom_non_h_electron_sum: 315<br />fill: red","count:  12645<br />local_res_atom_non_h_electron_sum: 317<br />fill: red","count:    428<br />local_res_atom_non_h_electron_sum: 320<br />fill: red","count:    142<br />local_res_atom_non_h_electron_sum: 326<br />fill: red","count:    100<br />local_res_atom_non_h_electron_sum: 332<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum: 335<br />fill: red","count:     18<br />local_res_atom_non_h_electron_sum: 336<br />fill: red","count:      4<br />local_res_atom_non_h_electron_sum: 338<br />fill: red","count:      3<br />local_res_atom_non_h_electron_sum: 341<br />fill: red","count:    166<br />local_res_atom_non_h_electron_sum: 344<br />fill: red","count:      5<br />local_res_atom_non_h_electron_sum: 348<br />fill: red","count:     12<br />local_res_atom_non_h_electron_sum: 349<br />fill: red","count:    542<br />local_res_atom_non_h_electron_sum: 350<br />fill: red","count:  15351<br />local_res_atom_non_h_electron_sum: 356<br />fill: red","count:     37<br />local_res_atom_non_h_electron_sum: 362<br />fill: red","count:   5712<br />local_res_atom_non_h_electron_sum: 364<br />fill: red","count:     36<br />local_res_atom_non_h_electron_sum: 367<br />fill: red","count:     67<br />local_res_atom_non_h_electron_sum: 368<br />fill: red","count:      4<br />local_res_atom_non_h_electron_sum: 372<br />fill: red","count:     85<br />local_res_atom_non_h_electron_sum: 374<br />fill: red","count:  13291<br />local_res_atom_non_h_electron_sum: 375<br />fill: red","count:    424<br />local_res_atom_non_h_electron_sum: 380<br />fill: red","count:     64<br />local_res_atom_non_h_electron_sum: 386<br />fill: red","count:    103<br />local_res_atom_non_h_electron_sum: 392<br />fill: red","count:      6<br />local_res_atom_non_h_electron_sum: 402<br />fill: red","count:      5<br />local_res_atom_non_h_electron_sum: 404<br />fill: red","count:   7994<br />local_res_atom_non_h_electron_sum: 410<br />fill: red"],"type":"bar","marker":{"autocolorscale":false,"color":"rgba(248,118,109,1)","line":{"width":1.88976377952756,"color":"transparent"}},"name":"red","legendgroup":"red","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":26.2283105022831,"r":7.30593607305936,"b":40.1826484018265,"l":60.6392694063927},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-15.85,431.85],"tickmode":"array","ticktext":["0","100","200","300","400"],"tickvals":[0,100,200,300,400],"categoryorder":"array","categoryarray":["0","100","200","300","400"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":"local_res_atom_non_h_electron_sum","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-9615.5,201925.5],"tickmode":"array","ticktext":["0","50000","100000","150000","200000"],"tickvals":[0,50000,100000,150000,200000],"categoryorder":"array","categoryarray":["0","50000","100000","150000","200000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":"count","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(51,51,51,1)","width":0.66417600664176,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895},"y":0.93503937007874},"annotations":[{"text":"fill","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"cloud":false},"source":"A","attrs":{"12f0a82146b":{"x":{},"fill":{},"type":"bar"}},"cur_data":"12f0a82146b","visdat":{"12f0a82146b":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->

# 11. Tabel� pokazuj�c� 10 klas z najwi�ksz� niezgodno�ci� liczby atom�w i liczby elektron�w

```r
diff <- data %>% select(res_name, local_res_atom_non_h_count, dict_atom_non_h_count, local_res_atom_non_h_electron_sum, dict_atom_non_h_electron_sum) %>%
  group_by(res_name) %>% summarize(atom_diff = sum(abs(local_res_atom_non_h_count - dict_atom_non_h_count)), electron_diff = sum(abs(local_res_atom_non_h_electron_sum - dict_atom_non_h_electron_sum)))

diff %>% select(-electron_diff) %>% arrange(desc(atom_diff)) %>% head(10) %>% prettyTable()
```

<!--html_preserve--><div id="htmlwidget-aa186246d072cfcfcb70" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-aa186246d072cfcfcb70">{"x":{"style":"bootstrap","filter":"none","data":[["NAG","CLA","1PE","MLY","NAP","COA","NAD","PG4","MAN","NDP"],[72847,54211,16370,13436,13291,11720,9063,8278,6790,6162]],"container":"<table class=\"table table-striped table-hover\">\n  <thead>\n    <tr>\n      <th>res_name<\/th>\n      <th>atom_diff<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"dom":"Bfrtip","buttons":["copy","csv","excel","pdf","print"],"scrollX":true,"columnDefs":[{"className":"dt-right","targets":1}],"order":[],"autoWidth":false,"orderClasses":false,"rowCallback":"function(row, data) {\n}"}},"evals":["options.rowCallback"],"jsHooks":[]}</script><!--/html_preserve-->

```r
diff %>% select(-atom_diff) %>% arrange(desc(electron_diff)) %>% head(10) %>% prettyTable()
```

<!--html_preserve--><div id="htmlwidget-c1a1b5007938619c5a1c" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-c1a1b5007938619c5a1c">{"x":{"style":"bootstrap","filter":"none","data":[["NAG","CLA","1PE","MLY","NAP","COA","NAD","PG4","MAN","NDP"],[582604,328794,111632,101097,89372,88063,60809,55984,54320,42025]],"container":"<table class=\"table table-striped table-hover\">\n  <thead>\n    <tr>\n      <th>res_name<\/th>\n      <th>electron_diff<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"dom":"Bfrtip","buttons":["copy","csv","excel","pdf","print"],"scrollX":true,"columnDefs":[{"className":"dt-right","targets":1}],"order":[],"autoWidth":false,"orderClasses":false,"rowCallback":"function(row, data) {\n}"}},"evals":["options.rowCallback"],"jsHooks":[]}</script><!--/html_preserve-->

```r
rm(diff)
```

# 12. Sekcj� pokazuj�c� rozk�ad warto�ci part_01 

```r
melted <- data %>% filter(part == "part_01") %>% select(shape_segments_count:density_Z_4_0) %>% melt
means <-  melted %>% group_by(variable) %>% summarise(mean=mean(value)) 
melted %>% ggplot(aes(value)) +
    geom_density() +
    geom_vline(data = means, aes(xintercept=mean), linetype="dashed", color = "red") +
    geom_text(data=means, mapping=aes(x=mean, y=0, label=signif(mean, digits = 4)), 
              size=3, angle=90, vjust= 1, hjust=0, color = "red"
      )+  
    facet_wrap(~variable, ncol = 4, scales = "free") +
    theme_bw()+
    theme(
        axis.title.x = element_blank(),
        axis.title.y = element_blank(),
        axis.text.x = element_text(angle=90)
      ) 
```

![](ZED_raport_files/figure-html/part_01-1.png)<!-- -->

```r
rm(melted, means)
```

# 13. Interaktywny wykres lub animacj�.
W pkt 8. i 10.



# 14.Sekcj� sprawdzaj�c� czy na podstawie warto�ci innych kolumn mo�na przewidzie�:

##liczb� elektron�w 
Posiadam ok 1mln przyk�ad�w podzielonych na n klas 

```r
n = data %>% select(local_res_atom_non_h_electron_sum) %>% distinct() %>% count()
n
```

```
## # A tibble: 1 x 1
##       n
##   <int>
## 1   176
```
Zbi�r testowy z�o�ony z 3% tych danych powinien by� wystarczaj�cy. Ka�da z klas b�dzie mia�a wi�c ok. k przyk�ad�w (zale�nie od rozk�adu)

```r
k = nrow(data) * 0.03 / n
k
```

```
##          n
## 1 190.3739
```


```r
idx <- createDataPartition(data$local_res_atom_non_h_electron_sum, p=0.03, list=F, times = 1)
test <- data.frame(data[idx,]) %>% select(-res_name, -local_res_atom_non_h_count)
train <- data.frame(data[-idx,]) %>% select(-res_name, -local_res_atom_non_h_count)

ctrl <- trainControl(
    method = "boot", number = 20)
 
fit <- train(local_res_atom_non_h_electron_sum ~ .,
             data = train,
             method = "lm",
             trControl = ctrl,
             metric = "Rsquared",
             maximize = TRUE)

fit
```

```
## Linear Regression 
## 
## 1083352 samples
##     120 predictor
## 
## No pre-processing
## Resampling: Bootstrapped (20 reps) 
## Summary of sample sizes: 1083352, 1083352, 1083352, 1083352, 1083352, 1083352, ... 
## Resampling results:
## 
##   RMSE      Rsquared   MAE     
##   64.43256  0.4826822  36.23728
## 
## Tuning parameter 'intercept' was held constant at a value of TRUE
```

```r
rfClasses <- predict(fit, newdata = test) %>% round %>% as.integer

rm(idx, train, test)
```

##liczb� atom�w 
Posiadam ok 1mln przyk�ad�w podzielonych na n klas 

```r
n = data %>% select(local_res_atom_non_h_count) %>% distinct() %>% count()
n
```

```
## # A tibble: 1 x 1
##       n
##   <int>
## 1    64
```
Zbi�r testowy z�o�ony z 1% tych danych powinien by� wystarczaj�cy. Ka�da z klas b�dzie mia�a wi�c ok. k przyk�ad�w (zale�nie od rozk�adu)

```r
k = nrow(data) * 0.01 / n
k
```

```
##          n
## 1 174.5094
```


```r
idx <- createDataPartition(data$local_res_atom_non_h_count, p=0.01, list=F, times = 1)
test <- data.frame(data[idx,]) %>% select(-res_name, -local_res_atom_non_h_electron_sum)
train <- data.frame(data[-idx,]) %>% select(-res_name, -local_res_atom_non_h_electron_sum)

ctrl <- trainControl(
    method = "boot", number = 20)

fit <- train(local_res_atom_non_h_count ~ .,
             data = train,
             method = "lm",
             trControl = ctrl,
             metric = "Rsquared",
             maximize = TRUE)

fit
```

```
## Linear Regression 
## 
## 1105691 samples
##     120 predictor
## 
## No pre-processing
## Resampling: Bootstrapped (20 reps) 
## Summary of sample sizes: 1105691, 1105691, 1105691, 1105691, 1105691, 1105691, ... 
## Resampling results:
## 
##   RMSE     Rsquared   MAE     
##   9.45046  0.4919647  5.384283
## 
## Tuning parameter 'intercept' was held constant at a value of TRUE
```

```r
rfClasses <- predict(fit, newdata = test)

rm(idx, train, test)
```

# 15. Sekcj� pr�buj�c� stworzy� klasyfikator przewiduj�cy warto�� atrybutu res_name 
Posiadam ok 1mln przyk�ad�w podzielonych na 50 klas
Zbi�r testowy z�o�ony z 1% tych danych powinien by� wystarczaj�cy. Ka�da z klas b�dzie mia�a wi�c ok. k przyk�ad�w (zale�nie od rozk�adu)

```r
k = nrow(data) * 0.01 / 50
k
```

```
## [1] 223.372
```


```r
data$res_name <- as.factor(data$res_name)

idx <- createDataPartition(data$res_name, p=0.03, list=F, times = 1)
test <- data.frame(data[idx,]) %>% select(-local_res_atom_non_h_electron_sum, -local_res_atom_non_h_count)
train <- data.frame(data[-idx,]) %>% select(-local_res_atom_non_h_electron_sum, -local_res_atom_non_h_count)

ctrl <- trainControl(
    method = "boot", number = 5)

fit <- train(res_name ~ .,
             data = train,
             method = "rf",
             trControl = ctrl,
             ntree = 10,
             metric = "Accuracy",
             maximize = TRUE
             )

fit
```

```
## Random Forest 
## 
## 1083330 samples
##     120 predictor
##      50 classes: '1PE', 'ACT', 'ACY', 'ADP', 'ATP', 'BR', 'CA', 'CD', 'CL', 'CLA', 'COA', 'CU', 'DMS', 'EDO', 'EPE', 'FAD', 'FE', 'FE2', 'FMN', 'FMT', 'GDP', 'GOL', 'HEC', 'HEM', 'IOD', 'K', 'MAN', 'MES', 'MG', 'MLY', 'MN', 'MPD', 'NA', 'NAD', 'NAG', 'NAP', 'NDP', 'NI', 'NO3', 'PEG', 'PG4', 'PGE', 'PLP', 'PO4', 'SAH', 'SEP', 'SF4', 'SO4', 'TRS', 'ZN' 
## 
## No pre-processing
## Resampling: Bootstrapped (5 reps) 
## Summary of sample sizes: 1083330, 1083330, 1083330, 1083330, 1083330 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa    
##     2   0.4175302  0.3674636
##    61   0.5322474  0.4937026
##   121   0.5432910  0.5061765
## 
## Accuracy was used to select the optimal model using the largest value.
## The final value used for the model was mtry = 121.
```

```r
rfClasses <- predict(fit, newdata = test)
confusionMatrix(data = rfClasses, test$res_name)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction  1PE  ACT  ACY  ADP  ATP   BR   CA   CD   CL  CLA  COA   CU
##        1PE   66    2    0    0    0    0    1    0    0    2    0    0
##        ACT    1  220    6    3    0    0    6    1   14    0    2    0
##        ACY    0    3   31    0    0    0    0    0    1    0    0    0
##        ADP    0    1    0  208   22    0    2    0    0    0    3    1
##        ATP    0    0    0   10  127    0    0    0    0    0    0    0
##        BR     0    0    0    0    0   70    5    1    3    0    0    0
##        CA     0    5    2    4    1   15 1214   25   52    0    0   14
##        CD     0    1    2    0    0    1    5  126    1    0    0    3
##        CL     3   18    4    1    0   32  100    5 1356    1    5    5
##        CLA    0    0    0    1    0    0    1    0    1  294    1    0
##        COA    0    0    0    2    0    0    0    0    0    1   66    0
##        CU     0    0    0    0    0    0    1    1    0    0    0   86
##        DMS    1   12    1    0    0    0    3    2    9    0    3    0
##        EDO   19  129   22    4    4   12   38    7   94    2   20    3
##        EPE    0    0    1    0    0    0    0    1    1    0    1    0
##        FAD    0    0    0    3    1    0    1    0    0    0    4    0
##        FE     0    0    1    0    0    0    3    0    0    0    0    1
##        FE2    0    0    0    0    0    0    3    2    0    0    0    0
##        FMN    0    0    0    0    0    0    0    0    0    0    0    0
##        FMT    0    5    0    0    1    0    1    0    3    0    0    0
##        GDP    0    0    0    9    3    0    0    0    0    0    0    0
##        GOL   34  135   26    8    6   14   58    9  123    6   20    1
##        HEC    0    0    0    0    0    0    0    0    1    0    0    0
##        HEM    1    5    0    4    1    0    3    1    6    2    2    0
##        IOD    0    5    0    0    0    9   16   15   18    0    2    2
##        K      0    0    1    0    0    4    8    0   14    0    0    0
##        MAN    0    0    0    0    0    0    2    0    2    0    0    0
##        MES    0    1    0    0    0    0    2    0    0    0    1    0
##        MG     2    5    1    2    0    3   44    4   33    0    1    3
##        MLY    2    4    2    1    0    0    0    2    0    1    3    0
##        MN     0    0    1    2    0    0   14    6    1    0    0    2
##        MPD    1    3    0    1    1    0    2    1    1    0    0    0
##        NA     2    5    1    1    0    1   29    2   44    1    1    0
##        NAD    0    0    0    6    2    0    1    0    1    0    6    0
##        NAG   26   12    5   36   19    1   32    6   25   31   27    2
##        NAP    1    0    0    5    0    0    0    0    0    0    2    1
##        NDP    0    1    0    0    1    0    2    0    0    0    3    0
##        NI     0    0    0    0    0    0    1    0    0    0    0    0
##        NO3    0    0    1    0    0    0    1    0    0    0    0    0
##        PEG    5    2    1    0    1    0    1    0    4    1    3    0
##        PG4    4    2    0    0    0    1    1    0    0    0    0    0
##        PGE    3    0    0    0    1    0    1    0    1    1    0    0
##        PLP    1    0    0    0    0    0    1    0    0    0    0    0
##        PO4    1   10    3    2    1    0   14    3   12    0    0    2
##        SAH    0    0    0    1    3    0    0    0    0    0    1    0
##        SEP    1    0    0    1    2    1    1    0    0    0    0    0
##        SF4    0    0    0    0    0    0    1    0    0    0    0    1
##        SO4    8  118   27   18    5   23  202   33  243    6   13   15
##        TRS    0    1    0    0    1    0    1    0    0    0    0    0
##        ZN     2    7    4    5    1    4   59   38    9    0    0   70
##           Reference
## Prediction  DMS  EDO  EPE  FAD   FE  FE2  FMN  FMT  GDP  GOL  HEC  HEM
##        1PE    0    6    0    1    0    0    1    1    0    2    0    1
##        ACT    7   28    3    0    1    2    1   10    0   38    3    9
##        ACY    0    1    0    0    0    0    0    1    0    5    0    2
##        ADP    1    3    1    8    0    0    1    0   21    4    0    1
##        ATP    0    0    2    1    0    0    2    0    8    0    1    2
##        BR     0    2    0    0    0    0    0    0    0    1    0    0
##        CA     6   22    1    1   13   19    1    5    0   30    3    5
##        CD     1    2    0    0    1    0    0    0    0    1    0    1
##        CL     8   75    6    0    8    5    1   10    0   76    2   12
##        CLA    0    1    1    0    0    0    0    0    0    1    0    4
##        COA    0    2    0    1    0    0    1    0    0    1    0    0
##        CU     0    0    0    0    0    1    0    0    0    0    0    0
##        DMS  302   12    4    0    2    0    1    3    0   11    0    1
##        EDO   32 1731   14   10    4    3    7   63    3  511   10   39
##        EPE    0    2   43    0    0    0    0    1    0    2    0    0
##        FAD    0    0    0  292    0    0    5    0    1    2    0    0
##        FE     0    1    0    0   37    0    0    0    0    0    0    1
##        FE2    0    1    0    0    1   49    1    0    0    0    0    2
##        FMN    0    0    0    3    0    0  110    0    2    1    1    1
##        FMT    1    9    1    0    0    0    1   69    0    7    0    2
##        GDP    0    0    1    1    0    0    2    0   88    0    0    1
##        GOL   46  462   23    9    5    8    7   56    4 2239    8   46
##        HEC    0    0    0    0    0    0    0    0    0    1  104    8
##        HEM    1   12    0    5    1    1    1    0    0    9   21  744
##        IOD    5    2    0    0    2    0    0    1    0    5    0    2
##        K      1    1    0    0    1    1    0    0    0    4    0    2
##        MAN    0    1    0    1    0    0    0    0    0    2    0    0
##        MES    1    0    5    0    0    0    0    0    0    4    0    1
##        MG     9   14    3    4    4    4    1    3    0   49    0    4
##        MLY    1    5    3    4    0    0    0    0    0   14    0    2
##        MN     0    0    0    1    4    1    0    0    0    2    0    1
##        MPD    3    6    0    0    0    0    0    2    0   10    0    0
##        NA     5   30    0    1    1    1    0    1    0   40    0    0
##        NAD    1    4    2    7    0    0    1    1    1    3    1    0
##        NAG    5   56   17   36    3    2   20    3    7  121    5   13
##        NAP    0    2    0    4    0    0    3    0    1    0    0    1
##        NDP    0    0    0    2    0    0    0    0    2    2    0    1
##        NI     0    0    0    1    0    0    0    1    0    0    0    1
##        NO3    3    0    1    0    0    0    0    0    0    2    0    3
##        PEG    3   14    3    0    0    0    1    1    0   27    1    2
##        PG4    0    5    2    1    0    0    0    2    0    7    1    3
##        PGE    0    0    0    0    0    0    0    1    0    2    0    0
##        PLP    0    0    2    0    0    0    0    0    0    0    0    0
##        PO4    3   10    3    0    4    3    1    3    1   13    0    7
##        SAH    0    1    0    2    0    0    1    0    2    0    0    0
##        SEP    0    0    0    1    0    0    1    0    0    0    0    0
##        SF4    0    0    0    0    0    0    0    0    0    0    0    1
##        SO4  142  151   29    9   25   21   12   16    1  289    9   55
##        TRS    0    0    0    0    0    0    0    0    0    1    0    0
##        ZN     4   11    0    0   27   20    1    2    0    9    0    5
##           Reference
## Prediction  IOD    K  MAN  MES   MG  MLY   MN  MPD   NA  NAD  NAG  NAP
##        1PE    0    0    0    1    2    1    1    0    0    0    5    0
##        ACT    2    2    3    3    6    2    1    3    8    6    8    3
##        ACY    0    0    0    1    0    0    0    0    1    0    1    0
##        ADP    0    0    0    0    3    2    1    0    0   10    6    2
##        ATP    0    0    0    0    1    0    0    0    0    1    2    0
##        BR     7    1    0    0    0    0    0    0    0    0    0    0
##        CA    28   38    2    3   64    4   52    3   40    2   14    0
##        CD     9    1    0    1    2    0    2    0    0    0    1    0
##        CL    30   91    4    3   73    3    9    3  157    3   11    3
##        CLA    0    1    1    0    1    0    0    0    0    0    6    0
##        COA    0    0    1    0    0    2    0    0    0    2    8    0
##        CU     0    0    0    0    1    0    4    0    0    0    0    0
##        DMS    2    3    1    2    7    1    0    0    5    0    3    0
##        EDO   15   12   20   24   75   28    4   32   78   18   87    9
##        EPE    0    0    0    4    1    1    0    0    0    1    0    0
##        FAD    0    0    1    1    1    2    0    0    1   14    8    1
##        FE     0    2    0    0    3    0    1    0    1    0    0    0
##        FE2    0    0    0    0    1    0    1    0    0    0    0    0
##        FMN    0    0    0    0    0    0    0    0    0    3    3    2
##        FMT    1    0    1    0    5    0    0    1    2    1    0    0
##        GDP    0    0    0    0    0    0    0    0    0    2    0    0
##        GOL   18   23   37   29  110   50   12   79  104   28  189   20
##        HEC    1    0    0    1    0    0    0    1    0    0    0    1
##        HEM    3    0    2    0    4    2    0    3    4    2    9    1
##        IOD  372    7    1    0    4    0    3    0    5    0    4    1
##        K      4  161    0    0    6    0    3    1    4    0    0    0
##        MAN    0    0   67    0    1    0    0    2    2    1   12    1
##        MES    0    3    1   77    3    0    0    0    0    0    2    1
##        MG     2   10    3    7  655    8   13   10   25    5   17    1
##        MLY    0    0    2    3    3  142    1    3    2    3    9    3
##        MN     3    2    0    0    6    1  148    0    3    0    1    0
##        MPD    1    0    1    1    3    2    1   65    1    2    2    1
##        NA     4    4    1    0   23    3    0    0  316    0    8    3
##        NAD    0    1    1    0    0    1    1    0    1  212    9   16
##        NAG    1   13   57   11   49   17    8   16   17   41 1690   29
##        NAP    0    0    0    0    1    1    0    1    0   11    4  178
##        NDP    0    0    1    0    0    0    0    0    0    5    3    9
##        NI     4    0    0    1    0    0    1    0    0    0    0    0
##        NO3    0    0    0    2    1    0    0    0    0    0    0    1
##        PEG    0    1    6    1    3    0    1    0    2    2    8    0
##        PG4    0    0    0    0    0    0    0    1    1    1    7    0
##        PGE    0    0    1    1    3    0    1    0    0    0    1    2
##        PLP    0    0    0    3    0    0    0    1    0    1    1    1
##        PO4    2    3    0   10    7    0    7    3    6    0    6    5
##        SAH    0    0    0    0    1    0    0    0    0    3    0    1
##        SEP    0    0    1    0    0    0    1    0    0    1    1    0
##        SF4    0    0    0    0    0    0    0    0    0    0    0    0
##        SO4   45   29   12   49  142   26   65   53   61   13   80   16
##        TRS    0    0    1    0    5    3    0    0    0    0    0    0
##        ZN    13   12    2    0   23    3   35    2    7    3    5    0
##           Reference
## Prediction  NDP   NI  NO3  PEG  PG4  PGE  PLP  PO4  SAH  SEP  SF4  SO4
##        1PE    0    0    0    8    7    2    0    0    0    1    0    3
##        ACT    1    0   10    8    7    1    0    7    0    0    0   44
##        ACY    0    0    0    0    0    0    0    0    1    0    0    0
##        ADP    2    0    0    0    0    0    0    4    4    1    0    2
##        ATP    0    0    0    1    0    0    0    0    3    0    0    0
##        BR     0    0    0    0    0    1    0    0    0    1    0    1
##        CA     0   13    2    3    0    1    1   20    3    2    0   77
##        CD     0    4    0    0    1    0    0    0    0    1    0    5
##        CL     2    1    2   11    3    1    2   19    0    2    1   83
##        CLA    1    0    0    0    1    0    0    0    0    0    0    2
##        COA    1    0    0    1    0    0    0    1    0    0    0    3
##        CU     0    1    0    0    0    0    0    0    0    0    0    0
##        DMS    1    0    1    1    2    0    1    7    1    0    0   27
##        EDO   11    2   18   90   33   34    4   23    4    0    0  176
##        EPE    0    0    0    0    1    0    0    1    1    0    0    2
##        FAD    1    0    0    1    1    0    1    0    0    0    0    0
##        FE     0    0    0    0    0    0    0    0    0    0    0    1
##        FE2    0    1    0    0    0    0    0    1    0    0    0    2
##        FMN    0    0    0    0    0    1    0    0    0    1    0    0
##        FMT    1    0    1    1    1    0    0    0    0    0    0    1
##        GDP    0    0    0    0    0    1    0    0    1    0    0    0
##        GOL   13    5   16  130   69   49    4   69    6    5    0  273
##        HEC    0    0    0    0    0    0    0    0    0    0    0    1
##        HEM    4    0    1    1    2    2    0    2    1    0    0   10
##        IOD    0    3    0    3    0    0    0    1    0    0    0   24
##        K      0    0    0    1    0    0    0    0    0    0    0    8
##        MAN    0    1    0    0    2    1    0    0    0    0    0    4
##        MES    0    0    0    3    0    0    2    2    0    1    0   13
##        MG     1    4    1    1    2    2    0    9    2    1    0   45
##        MLY    1    1    0    1    0    1    2    2    0    2    1    2
##        MN     0    3    0    0    1    0    0    3    0    0    0    8
##        MPD    1    0    1    1    2    0    1    3    1    0    0   15
##        NA     2    0    1    2    0    1    0    6    0    0    0   12
##        NAD   14    0    0    0    1    0    3    0    0    0    0    2
##        NAG   14    0    2   31   21   16    4   25   15    9    0   76
##        NAP   23    0    0    2    0    0    0    0    1    0    0    0
##        NDP   82    0    0    0    1    0    2    1    0    1    0    1
##        NI     0   38    0    1    1    0    0    0    0    1    0    1
##        NO3    0    0   57    0    0    1    0    0    0    0    0    0
##        PEG    0    0    0   80    4    4    0    9    0    0    0    9
##        PG4    0    0    0    8   61    3    0    1    0    0    0    2
##        PGE    0    0    0    3    3   29    1    0    0    0    0    5
##        PLP    0    1    0    0    0    1   97    0    0    0    0    2
##        PO4    1    2    0    2    2    0    2  307    1    4    0   95
##        SAH    0    0    0    0    0    0    0    0   91    2    0    0
##        SEP    1    0    0    0    0    0    0    1    1   76    0    1
##        SF4    0    0    0    0    0    0    0    0    0    0  145    0
##        SO4    9   34   27   31   10   12   16  446    4   15    1 3966
##        TRS    0    0    0    0    0    0    0    1    0    0    0    1
##        ZN     0   33    2    1    0    1    0   18    1    6    1   38
##           Reference
## Prediction  TRS   ZN
##        1PE    1    0
##        ACT    3    3
##        ACY    0    0
##        ADP    0    0
##        ATP    0    1
##        BR     0    1
##        CA     1  117
##        CD     0   15
##        CL     3   34
##        CLA    0    0
##        COA    0    0
##        CU     0   21
##        DMS    0    3
##        EDO   13   15
##        EPE    0    0
##        FAD    0    0
##        FE     0    2
##        FE2    0    6
##        FMN    0    1
##        FMT    0    1
##        GDP    0    0
##        GOL   51   31
##        HEC    0    2
##        HEM    0    2
##        IOD    0   12
##        K      0    1
##        MAN    1    1
##        MES    0    1
##        MG     1   23
##        MLY    0    3
##        MN     0   13
##        MPD    2    1
##        NA     0    8
##        NAD    0    0
##        NAG    8   40
##        NAP    1    0
##        NDP    0    0
##        NI     0    4
##        NO3    0    0
##        PEG    1    1
##        PG4    0    0
##        PGE    1    0
##        PLP    0    3
##        PO4    4   12
##        SAH    0    0
##        SEP    0    0
##        SF4    0    1
##        SO4   12  122
##        TRS   43    2
##        ZN     1 1269
## 
## Overall Statistics
##                                           
##                Accuracy : 0.5993          
##                  95% CI : (0.5941, 0.6046)
##     No Information Rate : 0.1504          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.567           
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: 1PE Class: ACT Class: ACY Class: ADP
## Sensitivity            0.358696   0.308989  0.2167832   0.615385
## Specificity            0.998531   0.991895  0.9994908   0.996806
## Pos Pred Value         0.573913   0.452675  0.6458333   0.662420
## Neg Pred Value         0.996469   0.985111  0.9966549   0.996086
## Prevalence             0.005488   0.021235  0.0042648   0.010081
## Detection Rate         0.001968   0.006561  0.0009245   0.006203
## Detection Prevalence   0.003430   0.014494  0.0014316   0.009365
## Balanced Accuracy      0.678613   0.650442  0.6081370   0.806096
##                      Class: ATP Class: BR Class: CA Class: CD Class: CL
## Sensitivity            0.622549  0.366492   0.64540  0.432990   0.65412
## Specificity            0.998950  0.999280   0.97744  0.998165   0.97040
## Pos Pred Value         0.783951  0.744681   0.62967  0.673797   0.59292
## Neg Pred Value         0.997692  0.996381   0.97889  0.995051   0.97705
## Prevalence             0.006084  0.005696   0.05610  0.008679   0.06183
## Detection Rate         0.003788  0.002088   0.03621  0.003758   0.04044
## Detection Prevalence   0.004831  0.002803   0.05750  0.005577   0.06821
## Balanced Accuracy      0.810749  0.682886   0.81142  0.715577   0.81226
##                      Class: CLA Class: COA Class: CU Class: DMS Class: EDO
## Sensitivity            0.842407   0.347368  0.405660   0.510998    0.64469
## Specificity            0.999277   0.999190  0.999100   0.995962    0.93921
## Pos Pred Value         0.924528   0.709677  0.741379   0.694253    0.48003
## Neg Pred Value         0.998344   0.996292  0.996229   0.991268    0.96812
## Prevalence             0.010409   0.005667  0.006323   0.017626    0.08008
## Detection Rate         0.008768   0.001968  0.002565   0.009007    0.05163
## Detection Prevalence   0.009484   0.002774  0.003460   0.012973    0.10755
## Balanced Accuracy      0.920842   0.673279  0.702380   0.753480    0.79195
##                      Class: EPE Class: FAD Class: FE Class: FE2 Class: FMN
## Sensitivity            0.252941   0.719212  0.256944   0.347518   0.594595
## Specificity            0.999371   0.998491  0.999491   0.999341   0.999430
## Pos Pred Value         0.671875   0.853801  0.685185   0.690141   0.852713
## Neg Pred Value         0.996205   0.996565  0.996804   0.997250   0.997755
## Prevalence             0.005070   0.012109  0.004295   0.004205   0.005517
## Detection Rate         0.001282   0.008709  0.001103   0.001461   0.003281
## Detection Prevalence   0.001909   0.010200  0.001610   0.002118   0.003847
## Balanced Accuracy      0.626156   0.858851  0.628218   0.673429   0.797012
##                      Class: FMT Class: GDP Class: GOL Class: HEC
## Sensitivity            0.269531   0.619718    0.63106   0.611765
## Specificity            0.998557   0.999371    0.91548   0.999490
## Pos Pred Value         0.589744   0.807339    0.46910   0.859504
## Neg Pred Value         0.994403   0.998384    0.95448   0.998024
## Prevalence             0.007635   0.004235    0.10582   0.005070
## Detection Rate         0.002058   0.002625    0.06678   0.003102
## Detection Prevalence   0.003489   0.003251    0.14235   0.003609
## Balanced Accuracy      0.634044   0.809545    0.77327   0.805628
##                      Class: HEM Class: IOD Class: K Class: MAN Class: MES
## Sensitivity             0.75456    0.65608 0.383333   0.290043   0.322176
## Specificity             0.99597    0.99539 0.998037   0.998889   0.998588
## Pos Pred Value          0.85029    0.70992 0.712389   0.644231   0.620968
## Neg Pred Value          0.99259    0.99409 0.992223   0.995094   0.995151
## Prevalence              0.02941    0.01691 0.012526   0.006889   0.007128
## Detection Rate          0.02219    0.01109 0.004802   0.001998   0.002296
## Detection Prevalence    0.02610    0.01563 0.006740   0.003102   0.003698
## Balanced Accuracy       0.87527    0.82574 0.690685   0.644466   0.660382
##                      Class: MG Class: MLY Class: MN Class: MPD Class: NA
## Sensitivity            0.50423   0.465574  0.392573   0.229682  0.370023
## Specificity            0.98802   0.997321  0.997617   0.997774  0.992533
## Pos Pred Value         0.62920   0.614719  0.651982   0.467626  0.564286
## Neg Pred Value         0.98018   0.995105  0.993124   0.993471  0.983682
## Prevalence             0.03874   0.009096  0.011244   0.008440  0.025470
## Detection Rate         0.01953   0.004235  0.004414   0.001939  0.009424
## Detection Prevalence   0.03105   0.006889  0.006770   0.004146  0.016701
## Balanced Accuracy      0.74613   0.731448  0.695095   0.613728  0.681278
##                      Class: NAD Class: NAG Class: NAP Class: NDP Class: NI
## Sensitivity            0.534005    0.75751   0.572347   0.438503  0.258503
## Specificity            0.997374    0.96709   0.998043   0.998860  0.999461
## Pos Pred Value         0.709030    0.62132   0.732510   0.683333  0.678571
## Neg Pred Value         0.994433    0.98244   0.996004   0.996857  0.996744
## Prevalence             0.011840    0.06654   0.009275   0.005577  0.004384
## Detection Rate         0.006323    0.05040   0.005309   0.002446  0.001133
## Detection Prevalence   0.008917    0.08112   0.007247   0.003579  0.001670
## Balanced Accuracy      0.765690    0.86230   0.785195   0.718682  0.628982
##                      Class: NO3 Class: PEG Class: PG4 Class: PGE
## Sensitivity            0.401408   0.187354   0.255230  0.1757576
## Specificity            0.999521   0.996315   0.998408  0.9990409
## Pos Pred Value         0.780822   0.396040   0.535088  0.4754098
## Neg Pred Value         0.997459   0.989588   0.994673  0.9959365
## Prevalence             0.004235   0.012735   0.007128  0.0049210
## Detection Rate         0.001700   0.002386   0.001819  0.0008649
## Detection Prevalence   0.002177   0.006024   0.003400  0.0018193
## Balanced Accuracy      0.700465   0.591834   0.626819  0.5873992
##                      Class: PLP Class: PO4 Class: SAH Class: SEP
## Sensitivity            0.678322   0.310415   0.640845   0.575758
## Specificity            0.999461   0.991703   0.999461   0.999521
## Pos Pred Value         0.843478   0.532062   0.834862   0.826087
## Neg Pred Value         0.998623   0.979304   0.998474   0.998325
## Prevalence             0.004265   0.029496   0.004235   0.003937
## Detection Rate         0.002893   0.009156   0.002714   0.002267
## Detection Prevalence   0.003430   0.017208   0.003251   0.002744
## Balanced Accuracy      0.838891   0.651059   0.820153   0.787639
##                      Class: SF4 Class: SO4 Class: TRS Class: ZN
## Sensitivity            0.973154     0.7864   0.292517   0.71614
## Specificity            0.999880     0.9017   0.999491   0.98473
## Pos Pred Value         0.973154     0.5862   0.716667   0.72349
## Neg Pred Value         0.999880     0.9598   0.996893   0.98417
## Prevalence             0.004444     0.1504   0.004384   0.05285
## Detection Rate         0.004324     0.1183   0.001282   0.03785
## Detection Prevalence   0.004444     0.2018   0.001789   0.05231
## Balanced Accuracy      0.986517     0.8441   0.646004   0.85043
```

```r
rm(idx, train, test)
```