# Handling Inventory Data in CLDF

In order to provide sound inventory data in CLDF format, we suggest to use [CLTS](https://clts.clld.org) as a reference catalog for sound inventory data. Similar to the handling of concept elicitation glosses that can be linked to the [Concepticon](https://concepticon.clld.org) project, sounds in your inventory data *can* be linked to CLTS or left empty, if no suitable sound can be found. 

An example dataset that can be used as a current best-practice example is available with the [allenbai](https://github.com/lexibank/allenbai) dataset, which offers lexical data and sound inventory data for 9 varieties of the Bai language spoken in Yunnan (China). 

Similar to the handling of concept lists, which can be submitted directly to our Concepticon reference catalogue, we suggest to add a sound inventory list to the CLTS project. Once this has been done, you can use the CLTS transcription system in your CLDFBench code to specify the CLTS equivalent for each sound, where this is available.

## Submitting Sound Inventory Lists to CLTS

If you submit a sound inventory list to the CLTS project, note that the amount of information this list should contain is restricted. This has been done on purpose, since all additional information that might be of interest should go directly into the CLDF dataset accompanying an inventory collection. 

In order to prepare your sound list for submission to CLTS, you should create a tab-separated file, called `graphemes.tsv`, and add it into a folder that has the name of your dataset (lowercase, no spaces, no numbers). Detailed instructions can be found with the [CLTS project](https://github.com/cldf-clts/clts/blob/master/CONTRIBUTING.md). What you should know, however, is that, your sound list should not provide detailed information on each language, but only represent each sound once, so there should be no duplicated lines in your sound list. 

As an example sound list, consider the one we prepared for [allenbai](https://github.com/cldf-clts/clts/blog/master/sources/allenbai/graphemes.tsv). 

## Preparing your CLDF dataset

If you prepare your CLDF dataset, you should retrieve the transcriptiondata from the CLTS project:

```python
from pyclts import CLTS

...

class Dataset(BaseDataset):
    ...
    def cmd_makecldf(self, args):
        clts = CLTS(args.clts.dir)
        bipa = clts.transcriptionsystem_dict['bipa']
        ab = clts.transcriptiondata_dict['allenbai']
```

Make sure to add the CLDF specs for your structure dataset:

```python

class Dataset(BaseDataset):
    ...
    def cldf_specs(self):
        return {
            None: BaseDataset.cldf_specs(self),
            'structure': CLDFSpec(
                module='StructureDataset',
                dir=self.cldf_dir,
                data_fnames={'ParameterTable': 'features.csv'}
            )
        }
```
If you now want to add your inventory data to the CLDF dataset, you should first add the CTLS columns to the ParameterTable:

```python
class Dataset(BaseDataset):
    ...
    def cmd_makecldf(self, args):
        ...
        with self.cldf_writer(args, cldf_spec='structure', clean=False) as writer:
            ...
            writer.cldf.add_columns(
                    'ParameterTable',
                    {'name': 'CLTS_BIPA', 'datatype': 'string'},
                    {'name': 'CLTS_Name', 'datatype': 'string'},
                    {
                        'name': 'Lexibank_BIPA',
                        'datatype': 'string',
                        'separator': ' '
                    },
                    {'name': 'Prosody', 'datatype': 'string'},
                    )
```

And from there you can now add the data for your inventories:

```python
class Dataset(BaseDataset):
    ...
    def cmd_makecldf(self, args):
        ...
        with self.cldf_writer(args, cldf_spec='structure', clean=False) as writer:
            ...
            for row in inventories:
                writer.objects['ParameterTable'].append({
                    'ID': row['ID'],
                    'Name': row['Value'],
                    'Description': name,
                    'CLTS_BIPA': ab.grapheme_map[row['Value']],
                    'CLTS_Name': bipa[ab.grapheme_map[row['Value']]] or ''
                    })
```


