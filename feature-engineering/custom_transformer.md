You can define Transformers in addition to the ones provided by GraphLab
Create by inheriting from TransformerBase. You must implement the __init__,
fit, and transform methods. 

How this is done is shown in the following example:

#### Introductory Example

```no-highlight
import graphlab
from graphlab.toolkits.feature_engineering import TransformerBase

class MyTransformer(TransformerBase):

    def __init__(self):
        pass

    def fit(self, dataset):
        ''' Learn means during the fit stage.'''
        self.mean = {}
        for col in dataset.column_names():
            self.mean[col] = dataset[col].mean()
        return self

    def transform(self, dataset):
        ''' Subtract means during the transform stage.'''
        for col in dataset.column_names():
            dataset[col] = dataset[col] - dataset[col].mean()
        return dataset


dataset = graphlab.SFrame({"a": range(100) })            
# Create the model
model = graphlab.feature_engineering.create(dataset, MyTransformer())

# Transform new data
transformed_sf = model.transform(dataset)

# Save and load this model.
model.save('foo-bar')
loaded_model = graphlab.load_model('foo-bar')
```

