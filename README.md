# Stone Data Challenge
## Sara Malvar

O case do Data Challenge da Stone se trata sobre estimar a probabilidade de fault (PD) de uma base de clientes da Stone. Essa é uma informação crucial para determinar se podemos ou não conceder crédito para um cliente e para que a Stone tome decisões sobre limite, taxa de juros etc. Este projeto é dividido em 4 partes: Manipulação dos dados e feature engineering, Visualização dos dados, Modelo final e Explicação dos resultados. 

Dentro dessas etapas, algumas variáveis foram removidas, outras foram criadas e funções de visualização também foram implementadas, permitindo que pudessemos entender quais subsegmentos possuem maiores clientes, risco de inadimplência, distribuição das variáveis etc, como podemos ver abaixo:

### Bibliotecas

As bibliotecas utilizadas durante os treinamentos ou testes foram:
```
from numpy.random import seed
seed(1)
import os
#from sklearn.model_selection import StratifiedKFold
#from sklearn.model_selection import RandomizedSearchCV, GridSearchCV
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.model_selection import train_test_split
from sklearn import preprocessing,metrics
from sklearn.ensemble import RandomForestClassifier
from sklearn.inspection import permutation_importance
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import confusion_matrix, f1_score, classification_report, accuracy_score, auc, roc_curve, roc_auc_score, precision_recall_curve
#from sklearn.utils import class_weight
#from imblearn.over_sampling import SMOTE
#from imblearn.over_sampling import ADASYN
from xgboost import XGBClassifier
from xgboost import plot_importance
from sklearn.model_selection import train_test_split
#from sklearn.model_selection import StratifiedShuffleSplit
import pickle
```

Além disso, sugiro fortemente que seja utilizado o Dask, uma biblioteca de computação em paralelo, devido ao tamanho do banco de dados. Para isso, instale as bibliotecas do Dask necessária (eles possuem uma API para cada classe do SKlearn) e defina seu *client*:

```

from dask.distributed import Client

client = Client(n_workers=4, threads_per_worker=1)
client

```


### Subsegmento (categórica)

![](figs/output1.png)

![](figs/output2.png)


### Valor emprestago (numérica)

![](figs/output3.png)

### Pagamento diário (numérica)

![](figs/output4.png)

### Divida total (numérica)
![](figs/output5.png)

## Técnicas utilizadas
Posteriormente, uma série de modelos foram testados como SVC, XGBoost, DecisionTree e RandomForest. Além disso, técnicas para lidar com dados desbalanceados como a SMOTE foram utilizadas e os hiperparâmetros também foram tunados, até chegarmos ao melhor modelo, como pode ser observado na tabela abaixo.



| Modelo       | AUC           | Recall  |
| ------------- |:-------------:| -----:|
| RandomForest SMOTE      | 76% | 85% |
| RandomForest SMORE + Undersampling      | 75%      |   80% |
| XGBoost |  75% | 74% |
| XGBoost SMOTE | 65% | 88%|
| **RandomForest** | **82%** |**80%**|

Além disso, utilizei o SHAP (SHapley Additive exPlanations) para identificar a importância de cada uma das features nos modelos. Todos sabemos que utilizar a `feature_importance` é extremamente *tricky*, até porque a definição de ganho de informação ou peso pode nos fornecer valores pouco confiáveis. É importante ressaltar que os valores de SHAP não fornecem causalidade, o que torna nossa análise ideal. Os resultados são mostrados abaixo:

![](figs/shap.png)

Desta análise, percebemos que algumas variáveis possuem um impacto relevante na saída do nosso modelo, como era de se esperar. Entre elas: valores baixos de `dias_ate_vencimento` e valores altos de `divida_total`. Ou seja, pessoas com dívidas totais altas a poucos dias do vencimento, tendem a ter um impacto maior na predição. Porém, algumas outras informações não são tão lógicas, como os subsegmentos. Observa-se que quando o subsegmento era "Alimentação rápida", "Bares e Restaurantes", "Comércio de alimentos", "Vestuário", "Oficinas automotivas", "Autopeças", "Salão de beleza" ou "Supermercados", o impacto no modelo também era maior. Talvez seja interessante ter sistemas de segurança extras quando a pessoa a requerer o crédito for de algum destes segmentos. 
