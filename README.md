## 概要

isucon13の練習として、private-isuを実施する
https://github.com/catatsuy/private-isu#private-isu

## 期日

10/17 3-shakeの感想戦まで

## 前提

golang使用

docker-compose.yaml mysqlについて、デフォルトのcpu1,memory:1gだと
初回ベンチマークがタイムアウトするため、cpu2,memory:2gのベンチマークを初期値とした。

# やったこと

## 1.初回ベンチ

```
❯ docker run --network host -i private-isu-benchmarker /opt/go/bin/benchmarker -t http://host.docker.internal -u /opt/go/userdata
{"pass":true,"score":2553,"success":2450,"fail":0,"messages":[]}
```

## 2.アクセスログとスロークエリの有効化と初回解析


### 2-1.アクセスログ解析

- ツール : kataribe
    - https://github.com/matsuu/kataribe

```
❯ cat access.log| kataribe
Top 20 Sort By Count
Count    Total    Mean  Stddev    Min  P50.0  P90.0  P95.0  P99.0    Max  2xx  3xx  4xx  5xx  TotalBytes   MinBytes  MeanBytes   MaxBytes  Request
  417    0.267  0.0006  0.0010  0.000  0.000  0.001  0.002  0.005  0.008  417    0    0    0      645933       1549       1549       1549  GET /css/style.css HTTP/1.1
  417    0.337  0.0008  0.0012  0.000  0.001  0.002  0.003  0.007  0.010  417    0    0    0      760608       1824       1824       1824  GET /js/main.js HTTP/1.1
  417    0.510  0.0012  0.0017  0.000  0.001  0.003  0.005  0.009  0.012  417    0    0    0      798555       1915       1915       1915  GET /js/timeago.min.js HTTP/1.1
  417    0.997  0.0024  0.0026  0.000  0.001  0.006  0.008  0.013  0.019  417    0    0    0       17931         43         43         43  GET /favicon.ico HTTP/1.1
  142  934.806  6.5831  2.2724  0.661  7.173  8.288  8.647  9.027  9.186  142    0    0    0     4765501       4007      33559      36358  GET / HTTP/1.1
  101    3.832  0.0379  0.0377  0.000  0.015  0.095  0.098  0.101  0.104    0  101    0    0           0          0          0          0  POST /login HTTP/1.1
   92    1.971  0.0214  0.0302  0.000  0.002  0.070  0.077  0.082  0.082   92    0    0    0     7669054      83332      83359      83379  GET /image/9985.jpg HTTP/1.1
   90    2.465  0.0274  0.0328  0.000  0.004  0.076  0.078  0.088  0.088   90    0    0    0     7805026      86700      86722      86739  GET /image/9986.jpg HTTP/1.1
   89    2.119  0.0238  0.0317  0.000  0.006  0.076  0.079  0.089  0.089   89    0    0    0    32732496     367761     367780     367807  GET /image/9987.jpg HTTP/1.1
   88    2.430  0.0276  0.0341  0.000  0.004  0.077  0.082  0.097  0.097   88    0    0    0     8640909      98174      98192      98205  GET /image/9983.jpg HTTP/1.1
   86    3.240  0.0377  0.0383  0.001  0.014  0.083  0.086  0.152  0.152   86    0    0    0    78763412     854674     915853     977027  GET /image/10001.png HTTP/1.1
   86    1.863  0.0217  0.0291  0.000  0.005  0.072  0.077  0.091  0.091   86    0    0    0     9252148     107553     107583     107599  GET /image/9989.jpg HTTP/1.1
   86    2.412  0.0280  0.0327  0.000  0.005  0.073  0.076  0.095  0.095   86    0    0    0     9601407     111615     111644     111662  GET /image/9988.jpg HTTP/1.1
   84    4.453  0.0530  0.0359  0.002  0.068  0.088  0.091  0.171  0.171   84    0    0    0    88863083    1057873    1057893    1057913  GET /image/10000.png HTTP/1.1
   84    1.390  0.0165  0.0278  0.000  0.002  0.071  0.076  0.098  0.098   84    0    0    0     5185002      61708      61726      61747  GET /image/9998.jpg HTTP/1.1
   84    2.292  0.0273  0.0344  0.000  0.006  0.077  0.086  0.108  0.108   84    0    0    0     8608968     102463     102487     102501  GET /image/9990.jpg HTTP/1.1
   84    3.546  0.0422  0.0321  0.000  0.059  0.074  0.076  0.107  0.107   84    0    0    0     7562365      90021      90028      90043  GET /image/9999.jpg HTTP/1.1
   84    2.734  0.0325  0.0341  0.000  0.004  0.076  0.078  0.097  0.097   84    0    0    0    10374130     123481     123501     123519  GET /image/9995.jpg HTTP/1.1
   84    1.626  0.0194  0.0290  0.000  0.004  0.075  0.077  0.091  0.091   84    0    0    0    14833959     176576     176594     176615  GET /image/9997.jpg HTTP/1.1
   84    1.977  0.0235  0.0343  0.000  0.003  0.076  0.080  0.149  0.149   84    0    0    0     7193816      85614      85640      85661  GET /image/9993.jpg HTTP/1.1

Top 20 Sort By Total
Count    Total    Mean  Stddev    Min  P50.0  P90.0  P95.0  P99.0    Max  2xx  3xx  4xx  5xx  TotalBytes   MinBytes  MeanBytes   MaxBytes  Request
  142  934.806  6.5831  2.2724  0.661  7.173  8.288  8.647  9.027  9.186  142    0    0    0     4765501       4007      33559      36358  GET / HTTP/1.1
    3   22.867  7.6223  0.1535  7.408  7.700  7.759  7.759  7.759  7.759    3    0    0    0      102072      34014      34024      34037  GET /posts?max_created_at=2016-01-02T11%3A46%3A24%2B09%3A00 HTTP/1.1
    3   22.058  7.3527  0.8646  6.392  7.178  8.488  8.488  8.488  8.488    3    0    0    0      101913      33953      33971      33991  GET /posts?max_created_at=2016-01-02T11%3A46%3A22%2B09%3A00 HTTP/1.1
    3   20.086  6.6953  0.2193  6.391  6.796  6.899  6.899  6.899  6.899    3    0    0    0      101754      33913      33918      33921  GET /posts?max_created_at=2016-01-02T11%3A46%3A27%2B09%3A00 HTTP/1.1
    2   15.768  7.8840  0.1850  7.699  8.069  8.069  8.069  8.069  8.069    2    0    0    0       67790      33864      33895      33926  GET /posts?max_created_at=2016-01-02T11%3A46%3A29%2B09%3A00 HTTP/1.1
    2   14.419  7.2095  0.1995  7.010  7.409  7.409  7.409  7.409  7.409    2    0    0    0       67791      33892      33895      33899  GET /posts?max_created_at=2016-01-02T11%3A46%3A25%2B09%3A00 HTTP/1.1
    2   13.849  6.9245  0.4375  6.487  7.362  7.362  7.362  7.362  7.362    2    0    0    0       67942      33944      33971      33998  GET /posts?max_created_at=2016-01-02T11%3A46%3A21%2B09%3A00 HTTP/1.1
    2    8.074  4.0370  0.4620  3.575  4.499  4.499  4.499  4.499  4.499    2    0    0    0       50553      24995      25276      25558  GET /@elva HTTP/1.1
    2    7.740  3.8700  0.5980  3.272  4.468  4.468  4.468  4.468  4.468    2    0    0    0       39790      19650      19895      20140  GET /@lesley HTTP/1.1
    1    6.281  6.2810  0.0000  6.281  6.281  6.281  6.281  6.281  6.281    1    0    0    0       30019      30019      30019      30019  GET /@angel HTTP/1.1
    1    5.708  5.7080  0.0000  5.708  5.708  5.708  5.708  5.708  5.708    1    0    0    0       25104      25104      25104      25104  GET /@brittany HTTP/1.1
    1    5.575  5.5750  0.0000  5.575  5.575  5.575  5.575  5.575  5.575    1    0    0    0       22998      22998      22998      22998  GET /@luann HTTP/1.1
    1    5.393  5.3930  0.0000  5.393  5.393  5.393  5.393  5.393  5.393    1    0    0    0       23317      23317      23317      23317  GET /@janelle HTTP/1.1
    1    5.211  5.2110  0.0000  5.211  5.211  5.211  5.211  5.211  5.211    1    0    0    0       33896      33896      33896      33896  GET /posts?max_created_at=2016-01-02T11%3A46%3A26%2B09%3A00 HTTP/1.1
    2    4.701  2.3505  0.2415  2.109  2.592  2.592  2.592  2.592  2.592    2    0    0    0       22190      11088      11095      11102  GET /@eileen HTTP/1.1
    1    4.639  4.6390  0.0000  4.639  4.639  4.639  4.639  4.639  4.639    1    0    0    0       22883      22883      22883      22883  GET /@antoinette HTTP/1.1
   84    4.453  0.0530  0.0359  0.002  0.068  0.088  0.091  0.171  0.171   84    0    0    0    88863083    1057873    1057893    1057913  GET /image/10000.png HTTP/1.1
    1    4.397  4.3970  0.0000  4.397  4.397  4.397  4.397  4.397  4.397    1    0    0    0       18444      18444      18444      18444  GET /@lynda HTTP/1.1
    1    4.381  4.3810  0.0000  4.381  4.381  4.381  4.381  4.381  4.381    1    0    0    0       21528      21528      21528      21528  GET /@karen HTTP/1.1
    1    4.378  4.3780  0.0000  4.378  4.378  4.378  4.378  4.378  4.378    1    0    0    0       23195      23195      23195      23195  GET /@carlene HTTP/1.1

Top 20 Sort By Mean
Count    Total    Mean  Stddev    Min  P50.0  P90.0  P95.0  P99.0    Max  2xx  3xx  4xx  5xx  TotalBytes   MinBytes  MeanBytes   MaxBytes  Request
    2   15.768  7.8840  0.1850  7.699  8.069  8.069  8.069  8.069  8.069    2    0    0    0       67790      33864      33895      33926  GET /posts?max_created_at=2016-01-02T11%3A46%3A29%2B09%3A00 HTTP/1.1
    3   22.867  7.6223  0.1535  7.408  7.700  7.759  7.759  7.759  7.759    3    0    0    0      102072      34014      34024      34037  GET /posts?max_created_at=2016-01-02T11%3A46%3A24%2B09%3A00 HTTP/1.1
    3   22.058  7.3527  0.8646  6.392  7.178  8.488  8.488  8.488  8.488    3    0    0    0      101913      33953      33971      33991  GET /posts?max_created_at=2016-01-02T11%3A46%3A22%2B09%3A00 HTTP/1.1
    2   14.419  7.2095  0.1995  7.010  7.409  7.409  7.409  7.409  7.409    2    0    0    0       67791      33892      33895      33899  GET /posts?max_created_at=2016-01-02T11%3A46%3A25%2B09%3A00 HTTP/1.1
    2   13.849  6.9245  0.4375  6.487  7.362  7.362  7.362  7.362  7.362    2    0    0    0       67942      33944      33971      33998  GET /posts?max_created_at=2016-01-02T11%3A46%3A21%2B09%3A00 HTTP/1.1
    3   20.086  6.6953  0.2193  6.391  6.796  6.899  6.899  6.899  6.899    3    0    0    0      101754      33913      33918      33921  GET /posts?max_created_at=2016-01-02T11%3A46%3A27%2B09%3A00 HTTP/1.1
  142  934.806  6.5831  2.2724  0.661  7.173  8.288  8.647  9.027  9.186  142    0    0    0     4765501       4007      33559      36358  GET / HTTP/1.1
    1    6.281  6.2810  0.0000  6.281  6.281  6.281  6.281  6.281  6.281    1    0    0    0       30019      30019      30019      30019  GET /@angel HTTP/1.1
    1    5.708  5.7080  0.0000  5.708  5.708  5.708  5.708  5.708  5.708    1    0    0    0       25104      25104      25104      25104  GET /@brittany HTTP/1.1
    1    5.575  5.5750  0.0000  5.575  5.575  5.575  5.575  5.575  5.575    1    0    0    0       22998      22998      22998      22998  GET /@luann HTTP/1.1
    1    5.393  5.3930  0.0000  5.393  5.393  5.393  5.393  5.393  5.393    1    0    0    0       23317      23317      23317      23317  GET /@janelle HTTP/1.1
    1    5.211  5.2110  0.0000  5.211  5.211  5.211  5.211  5.211  5.211    1    0    0    0       33896      33896      33896      33896  GET /posts?max_created_at=2016-01-02T11%3A46%3A26%2B09%3A00 HTTP/1.1
    1    4.639  4.6390  0.0000  4.639  4.639  4.639  4.639  4.639  4.639    1    0    0    0       22883      22883      22883      22883  GET /@antoinette HTTP/1.1
    1    4.397  4.3970  0.0000  4.397  4.397  4.397  4.397  4.397  4.397    1    0    0    0       18444      18444      18444      18444  GET /@lynda HTTP/1.1
    1    4.381  4.3810  0.0000  4.381  4.381  4.381  4.381  4.381  4.381    1    0    0    0       21528      21528      21528      21528  GET /@karen HTTP/1.1
    1    4.378  4.3780  0.0000  4.378  4.378  4.378  4.378  4.378  4.378    1    0    0    0       23195      23195      23195      23195  GET /@carlene HTTP/1.1
    1    4.185  4.1850  0.0000  4.185  4.185  4.185  4.185  4.185  4.185    1    0    0    0       22975      22975      22975      22975  GET /@gena HTTP/1.1
    1    4.112  4.1120  0.0000  4.112  4.112  4.112  4.112  4.112  4.112    1    0    0    0       19588      19588      19588      19588  GET /@amber HTTP/1.1
    1    4.084  4.0840  0.0000  4.084  4.084  4.084  4.084  4.084  4.084    1    0    0    0       18462      18462      18462      18462  GET /@angela HTTP/1.1
    2    8.074  4.0370  0.4620  3.575  4.499  4.499  4.499  4.499  4.499    2    0    0    0       50553      24995      25276      25558  GET /@elva HTTP/1.1

Top 20 Sort By Standard Deviation
Count    Total    Mean  Stddev    Min  P50.0  P90.0  P95.0  P99.0    Max  2xx  3xx  4xx  5xx  TotalBytes   MinBytes  MeanBytes   MaxBytes  Request
  142  934.806  6.5831  2.2724  0.661  7.173  8.288  8.647  9.027  9.186  142    0    0    0     4765501       4007      33559      36358  GET / HTTP/1.1
    3   22.058  7.3527  0.8646  6.392  7.178  8.488  8.488  8.488  8.488    3    0    0    0      101913      33953      33971      33991  GET /posts?max_created_at=2016-01-02T11%3A46%3A22%2B09%3A00 HTTP/1.1
    2    7.740  3.8700  0.5980  3.272  4.468  4.468  4.468  4.468  4.468    2    0    0    0       39790      19650      19895      20140  GET /@lesley HTTP/1.1
    2    8.074  4.0370  0.4620  3.575  4.499  4.499  4.499  4.499  4.499    2    0    0    0       50553      24995      25276      25558  GET /@elva HTTP/1.1
    2   13.849  6.9245  0.4375  6.487  7.362  7.362  7.362  7.362  7.362    2    0    0    0       67942      33944      33971      33998  GET /posts?max_created_at=2016-01-02T11%3A46%3A21%2B09%3A00 HTTP/1.1
    2    4.701  2.3505  0.2415  2.109  2.592  2.592  2.592  2.592  2.592    2    0    0    0       22190      11088      11095      11102  GET /@eileen HTTP/1.1
    3   20.086  6.6953  0.2193  6.391  6.796  6.899  6.899  6.899  6.899    3    0    0    0      101754      33913      33918      33921  GET /posts?max_created_at=2016-01-02T11%3A46%3A27%2B09%3A00 HTTP/1.1
    2   14.419  7.2095  0.1995  7.010  7.409  7.409  7.409  7.409  7.409    2    0    0    0       67791      33892      33895      33899  GET /posts?max_created_at=2016-01-02T11%3A46%3A25%2B09%3A00 HTTP/1.1
    2   15.768  7.8840  0.1850  7.699  8.069  8.069  8.069  8.069  8.069    2    0    0    0       67790      33864      33895      33926  GET /posts?max_created_at=2016-01-02T11%3A46%3A29%2B09%3A00 HTTP/1.1
    2    0.996  0.4980  0.1800  0.318  0.678  0.678  0.678  0.678  0.678    2    0    0    0       11198       5596       5599       5602  GET /posts/1341 HTTP/1.1
    3   22.867  7.6223  0.1535  7.408  7.700  7.759  7.759  7.759  7.759    3    0    0    0      102072      34014      34024      34037  GET /posts?max_created_at=2016-01-02T11%3A46%3A24%2B09%3A00 HTTP/1.1
   22    2.266  0.1030  0.1096  0.004  0.078  0.255  0.278  0.467  0.467    0   11   11    0           0          0          0          0  POST / HTTP/1.1
    2    0.760  0.3800  0.1040  0.276  0.484  0.484  0.484  0.484  0.484    2    0    0    0        7698       3674       3849       4024  GET /posts/9110 HTTP/1.1
    2    0.178  0.0890  0.0890  0.000  0.178  0.178  0.178  0.178  0.178    2    0    0    0      212827     106405     106413     106422  GET /image/5997.jpg HTTP/1.1
    2    0.182  0.0910  0.0870  0.004  0.178  0.178  0.178  0.178  0.178    2    0    0    0      682635     341306     341317     341329  GET /image/5193.jpg HTTP/1.1
    2    0.172  0.0860  0.0850  0.001  0.171  0.171  0.171  0.171  0.171    2    0    0    0      338434     169198     169217     169236  GET /image/3176.jpg HTTP/1.1
   14    1.128  0.0806  0.0674  0.003  0.088  0.118  0.277  0.277  0.277    0   14    0    0           0          0          0          0  POST /comment HTTP/1.1
    2    0.253  0.1265  0.0655  0.061  0.192  0.192  0.192  0.192  0.192    2    0    0    0      114622      57300      57311      57322  GET /image/1815.jpg HTTP/1.1
    2    0.262  0.1310  0.0650  0.066  0.196  0.196  0.196  0.196  0.196    2    0    0    0      162354      81174      81177      81180  GET /image/2938.jpg HTTP/1.1
   11    1.123  0.1021  0.0639  0.008  0.107  0.190  0.200  0.200  0.200    0   11    0    0           0          0          0          0  POST /register HTTP/1.1

Top 20 Sort By Maximum(100 Percentile)
Count    Total    Mean  Stddev    Min  P50.0  P90.0  P95.0  P99.0    Max  2xx  3xx  4xx  5xx  TotalBytes   MinBytes  MeanBytes   MaxBytes  Request
  142  934.806  6.5831  2.2724  0.661  7.173  8.288  8.647  9.027  9.186  142    0    0    0     4765501       4007      33559      36358  GET / HTTP/1.1
    3   22.058  7.3527  0.8646  6.392  7.178  8.488  8.488  8.488  8.488    3    0    0    0      101913      33953      33971      33991  GET /posts?max_created_at=2016-01-02T11%3A46%3A22%2B09%3A00 HTTP/1.1
    2   15.768  7.8840  0.1850  7.699  8.069  8.069  8.069  8.069  8.069    2    0    0    0       67790      33864      33895      33926  GET /posts?max_created_at=2016-01-02T11%3A46%3A29%2B09%3A00 HTTP/1.1
    3   22.867  7.6223  0.1535  7.408  7.700  7.759  7.759  7.759  7.759    3    0    0    0      102072      34014      34024      34037  GET /posts?max_created_at=2016-01-02T11%3A46%3A24%2B09%3A00 HTTP/1.1
    2   14.419  7.2095  0.1995  7.010  7.409  7.409  7.409  7.409  7.409    2    0    0    0       67791      33892      33895      33899  GET /posts?max_created_at=2016-01-02T11%3A46%3A25%2B09%3A00 HTTP/1.1
    2   13.849  6.9245  0.4375  6.487  7.362  7.362  7.362  7.362  7.362    2    0    0    0       67942      33944      33971      33998  GET /posts?max_created_at=2016-01-02T11%3A46%3A21%2B09%3A00 HTTP/1.1
    3   20.086  6.6953  0.2193  6.391  6.796  6.899  6.899  6.899  6.899    3    0    0    0      101754      33913      33918      33921  GET /posts?max_created_at=2016-01-02T11%3A46%3A27%2B09%3A00 HTTP/1.1
    1    6.281  6.2810  0.0000  6.281  6.281  6.281  6.281  6.281  6.281    1    0    0    0       30019      30019      30019      30019  GET /@angel HTTP/1.1
    1    5.708  5.7080  0.0000  5.708  5.708  5.708  5.708  5.708  5.708    1    0    0    0       25104      25104      25104      25104  GET /@brittany HTTP/1.1
    1    5.575  5.5750  0.0000  5.575  5.575  5.575  5.575  5.575  5.575    1    0    0    0       22998      22998      22998      22998  GET /@luann HTTP/1.1
    1    5.393  5.3930  0.0000  5.393  5.393  5.393  5.393  5.393  5.393    1    0    0    0       23317      23317      23317      23317  GET /@janelle HTTP/1.1
    1    5.211  5.2110  0.0000  5.211  5.211  5.211  5.211  5.211  5.211    1    0    0    0       33896      33896      33896      33896  GET /posts?max_created_at=2016-01-02T11%3A46%3A26%2B09%3A00 HTTP/1.1
    1    4.639  4.6390  0.0000  4.639  4.639  4.639  4.639  4.639  4.639    1    0    0    0       22883      22883      22883      22883  GET /@antoinette HTTP/1.1
    2    8.074  4.0370  0.4620  3.575  4.499  4.499  4.499  4.499  4.499    2    0    0    0       50553      24995      25276      25558  GET /@elva HTTP/1.1
    2    7.740  3.8700  0.5980  3.272  4.468  4.468  4.468  4.468  4.468    2    0    0    0       39790      19650      19895      20140  GET /@lesley HTTP/1.1
    1    4.397  4.3970  0.0000  4.397  4.397  4.397  4.397  4.397  4.397    1    0    0    0       18444      18444      18444      18444  GET /@lynda HTTP/1.1
    1    4.381  4.3810  0.0000  4.381  4.381  4.381  4.381  4.381  4.381    1    0    0    0       21528      21528      21528      21528  GET /@karen HTTP/1.1
    1    4.378  4.3780  0.0000  4.378  4.378  4.378  4.378  4.378  4.378    1    0    0    0       23195      23195      23195      23195  GET /@carlene HTTP/1.1
    1    4.185  4.1850  0.0000  4.185  4.185  4.185  4.185  4.185  4.185    1    0    0    0       22975      22975      22975      22975  GET /@gena HTTP/1.1
    1    4.112  4.1120  0.0000  4.112  4.112  4.112  4.112  4.112  4.112    1    0    0    0       19588      19588      19588      19588  GET /@amber HTTP/1.1

TOP 37 Slow Requests
 1  9.186  GET / HTTP/1.1
 2  9.027  GET / HTTP/1.1
 3  8.788  GET / HTTP/1.1
 4  8.764  GET / HTTP/1.1
 5  8.764  GET / HTTP/1.1
 6  8.760  GET / HTTP/1.1
 7  8.751  GET / HTTP/1.1
 8  8.647  GET / HTTP/1.1
 9  8.589  GET / HTTP/1.1
10  8.502  GET / HTTP/1.1
11  8.488  GET /posts?max_created_at=2016-01-02T11%3A46%3A22%2B09%3A00 HTTP/1.1
12  8.373  GET / HTTP/1.1
13  8.373  GET / HTTP/1.1
14  8.371  GET / HTTP/1.1
15  8.293  GET / HTTP/1.1
16  8.288  GET / HTTP/1.1
17  8.273  GET / HTTP/1.1
18  8.270  GET / HTTP/1.1
19  8.270  GET / HTTP/1.1
20  8.267  GET / HTTP/1.1
21  8.240  GET / HTTP/1.1
22  8.209  GET / HTTP/1.1
23  8.189  GET / HTTP/1.1
24  8.178  GET / HTTP/1.1
25  8.178  GET / HTTP/1.1
26  8.174  GET / HTTP/1.1
27  8.162  GET / HTTP/1.1
28  8.097  GET / HTTP/1.1
29  8.089  GET / HTTP/1.1
30  8.069  GET /posts?max_created_at=2016-01-02T11%3A46%3A29%2B09%3A00 HTTP/1.1
31  8.065  GET / HTTP/1.1
32  8.047  GET / HTTP/1.1
33  8.044  GET / HTTP/1.1
34  7.967  GET / HTTP/1.1
35  7.918  GET / HTTP/1.1
36  7.914  GET / HTTP/1.1
37  7.892  GET / HTTP/1.1
```

### 2-2.スロークエリログ解析

- ツール : percona-toolkit/pt-query-digest
    - https://docs.percona.com/percona-toolkit/


```
# 560ms user time, 20ms system time, 81.14M rss, 390.11G vsz
# Current date: Sat Sep 30 23:25:25 2023
# Hostname: NQ4TRW3RH6.local
# Files: ./private-isu-slow.log
# Overall: 2.15k total, 18 unique, 12.77 QPS, 2.44x concurrency __________
# Time range: 2023-09-30T12:44:48 to 2023-09-30T12:47:36
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# Exec time           410s   100ms   634ms   191ms   308ms    72ms   189ms
# Lock time            4ms       0   451us     1us     2us    10us     1us
# Rows sent        181.76k       0   9.77k   86.73    8.91  871.02    2.90
# Rows examine     201.49M       0  97.68k  96.14k  97.04k  11.47k  97.04k
# Query size         9.43M      38   1.63M   4.50k   80.10  68.43k   80.10

# Profile
# Rank Query ID                            Response time  Calls R/Call V/M
# ==== =================================== ============== ===== ====== ===
#    1 0x624863D30DAC59FA16849282195BE09F  334.1057 81.4%  1644 0.2032  0.02 SELECT comments
#    2 0x422390B42D4DD86C7539A5F45EB76A80   38.7242  9.4%   314 0.1233  0.01 SELECT comments
#    3 0x100EC8B5C400F34381F9D7F7FA80A53D   29.5557  7.2%   139 0.2126  0.03 SELECT comments
#    4 0x4858CF4D8CAA743E839C127C71B69E75    1.9424  0.5%    15 0.1295  0.01 SELECT posts
#    5 0xC37F2207FE2E699A3A976F5EBE87A97C    1.0845  0.3%     8 0.1356  0.01 SELECT comments
# MISC 0xMISC                                5.0481  1.2%    26 0.1942   0.0 <13 ITEMS>

# Query 1: 24.91 QPS, 5.06x concurrency, ID 0x624863D30DAC59FA16849282195BE09F at byte 9028935
# Scores: V/M = 0.02
# Time range: 2023-09-30T12:46:30 to 2023-09-30T12:47:36
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count         76    1644
# Exec time     81    334s   100ms   506ms   203ms   308ms    69ms   198ms
# Lock time     82     3ms       0   451us     1us     2us    11us     1us
# Rows sent      2   4.30k       0       3    2.68    2.90    0.90    2.90
# Rows examine  77 156.79M  97.66k  97.67k  97.66k  97.04k       0  97.04k
# Query size     1 131.86k      79      83   82.13   80.10    0.16   80.10
# String:
# Databases    isuconp
# Hosts        webapp-app-1.webapp_default
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms
# 100ms  ################################################################
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS FROM `isuconp` LIKE 'comments'\G
#    SHOW CREATE TABLE `isuconp`.`comments`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT * FROM `comments` WHERE `post_id` = 9986 ORDER BY `created_at` DESC LIMIT 3\G

# Query 2: 4.83 QPS, 0.60x concurrency, ID 0x422390B42D4DD86C7539A5F45EB76A80 at byte 8977880
# Scores: V/M = 0.01
# Time range: 2023-09-30T12:46:30 to 2023-09-30T12:47:35
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count         14     314
# Exec time      9     39s   100ms   288ms   123ms   189ms    34ms   105ms
# Lock time      9   342us       0    46us     1us     1us     2us     1us
# Rows sent      0     314       1       1       1       1       0       1
# Rows examine  14  29.95M  97.66k  97.66k  97.66k  97.04k       0  97.04k
# Query size     0  19.97k      64      66   65.11   65.89       1   62.76
# String:
# Databases    isuconp
# Hosts        webapp-app-1.webapp_default
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms
# 100ms  ################################################################
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS FROM `isuconp` LIKE 'comments'\G
#    SHOW CREATE TABLE `isuconp`.`comments`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT COUNT(*) AS `count` FROM `comments` WHERE `post_id` = 9990\G

# Query 3: 2.24 QPS, 0.48x concurrency, ID 0x100EC8B5C400F34381F9D7F7FA80A53D at byte 10280978
# Scores: V/M = 0.03
# Time range: 2023-09-30T12:46:33 to 2023-09-30T12:47:35
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count          6     139
# Exec time      7     30s   102ms   500ms   213ms   374ms    76ms   198ms
# Lock time      6   225us       0    61us     1us     1us     5us     1us
# Rows sent      0   1.35k       0      18    9.97   14.52    3.29    9.83
# Rows examine   6  13.26M  97.66k  97.68k  97.67k  97.04k       0  97.04k
# Query size     0  10.04k      73      75   73.97   72.65       0   72.65
# String:
# Databases    isuconp
# Hosts        webapp-app-1.webapp_default
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms
# 100ms  ################################################################
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS FROM `isuconp` LIKE 'comments'\G
#    SHOW CREATE TABLE `isuconp`.`comments`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT * FROM `comments` WHERE `post_id` = 8619 ORDER BY `created_at` DESC\G

# Query 4: 0.25 QPS, 0.03x concurrency, ID 0x4858CF4D8CAA743E839C127C71B69E75 at byte 7767391
# Scores: V/M = 0.01
# Time range: 2023-09-30T12:46:30 to 2023-09-30T12:47:30
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count          0      15
# Exec time      0      2s   100ms   182ms   129ms   180ms    32ms   105ms
# Lock time      0    13us       0     2us       0     1us       0     1us
# Rows sent     80 146.53k   9.77k   9.77k   9.77k   9.33k       0   9.33k
# Rows examine   0 293.07k  19.53k  19.54k  19.54k  19.40k       0  19.40k
# Query size     0   1.35k      92      92      92      92       0      92
# String:
# Databases    isuconp
# Hosts        webapp-app-1.webapp_default
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms
# 100ms  ################################################################
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS FROM `isuconp` LIKE 'posts'\G
#    SHOW CREATE TABLE `isuconp`.`posts`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT `id`, `user_id`, `body`, `mime`, `created_at` FROM `posts` ORDER BY `created_at` DESC\G

# Query 5: 0.13 QPS, 0.02x concurrency, ID 0xC37F2207FE2E699A3A976F5EBE87A97C at byte 7772550
# Scores: V/M = 0.01
# Time range: 2023-09-30T12:46:32 to 2023-09-30T12:47:33
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count          0       8
# Exec time      0      1s   101ms   186ms   136ms   180ms    33ms   144ms
# Lock time      0     8us       0     2us     1us     1us       0     1us
# Rows sent      0       8       1       1       1       1       0       1
# Rows examine   0 781.27k  97.66k  97.66k  97.66k  97.04k       0  97.04k
# Query size     0     864      83     143     108  136.99   15.93  107.46
# String:
# Databases    isuconp
# Hosts        webapp-app-1.webapp_default
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms
# 100ms  ################################################################
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS FROM `isuconp` LIKE 'comments'\G
#    SHOW CREATE TABLE `isuconp`.`comments`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT COUNT(*) AS count FROM `comments` WHERE `post_id` IN (442, 1717, 4365, 5022, 6720, 6753, 7523, 7901)\G
```