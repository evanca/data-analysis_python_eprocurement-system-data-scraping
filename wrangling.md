
### NOTE: Code outputs and email examples are hidden here due to ethical and legal reasoning.

You will see "name@*****.com" instead of the actual value "name@example.com". Personal data is subject to the protection requirements set out in The General Data Protection Regulation (GDPR).


```python
import pandas as pd
import numpy as np
import warnings
```

## Step 1. Identifying the objectives.

Client will use company name ("title"), email address ("rekviziti_email"), contact name ("kontaktpersona_name") & contact email ("kontaktpersona_mail") data for an email campaign.

- As the minimum, they need **at least one email address** to launch a campaign for the specific company.
- Primary email is the contact email ("kontaktpersona_mail"). If there is no contact email, they will use a company's email ("rekviziti_email").
- Company title is for informational purposes only and can be left "as is".
- Name and surname should be cleaned when possible.


```python
raw = pd.read_csv("raw_data.csv")
df = raw.copy()
df
```


```python
df.info()
```

## Step 2. Data assessment.

**Data tidiness checklist:**

- [x] Each variable forms a column
- [x] Each observation forms a row
- [x] Each type of observational unit forms a table

**Data quality issues (from visual assessment):**

- [] Null objects
- [] Wrong file category (should be "Bad quality" if contact email is Null)
- [] Unnecessary whitespace characters (e.g. "\t", "\n", "\r"...) 
- [] Incorrect OCR results - wrong characters instead of "@" (e.g. "(Ž", "Ģ", "(G)"...)
- [] Too long / messy email values (e.g. "CCA.al$ttkaeCatoja.byWa"IUaDskumentarekvizītus...")
- [] Phone nr. before email (e.g. "Tālruņanumurs12345678Evija.Perkone@*****.lv")
- [] Wrong contact names (e.g. "kas rīkojas", "parakstot šo", "(paraksts) mH!"...)
- [] Empty email containing just ".lv"
- [] Latvian characters in email (e.g. "saimniecība@*****.lv"...)
- [] Dash and underscore characters in email (e.g. "—_ieva.libkena@*****.lv"...)
- [] Comma instead of a dot in email
- [] No contact email
- [] Duplicates

## Step 3. Programmatic data cleaning.

### 3a. Null objects
If both company email and contact email are Null, there is no use of this entry for the email campaign. If only one is Null thought, we will leave it as is.


```python
len(df.query("rekviziti_email != rekviziti_email & kontaktpersona_mail != kontaktpersona_mail"))
```


```python
df = df.query("rekviziti_email == rekviziti_email or kontaktpersona_mail == kontaktpersona_mail")
if len(df.query("rekviziti_email != rekviziti_email & kontaktpersona_mail != kontaktpersona_mail")) == 0:
    print("Done")
```

### 3b. Wrong file category
File category should be "Bad quality" if contact email is Null.


```python
len(df.query("kontaktpersona_mail != kontaktpersona_mail & file_category == 'Scanned'"))
```


```python
pd.options.mode.chained_assignment = None # to remove false positive SettingWithCopyWarning

df.loc[df.kontaktpersona_mail.isnull(), 'file_category'] = 'Bad quality'
if len(df.query("kontaktpersona_mail != kontaktpersona_mail & file_category == 'Scanned'")) == 0:
    print("Done")
```

### 3c. Unnecessary whitespace сharacters


```python
df.loc[(df.filename == "AIZPND2.doc") | (df.filename == "JELGPPA2.pdf")] # test sample
```


```python
# \s matches any whitespace character, this is equivalent to the set [ \t\n\r\f\v]
for column in ["rekviziti_email", "kontaktpersona_mail"]:
    df[column] = df[column].str.replace('\s', '', regex=True)

# Doing the same for the title, but without removing the space character:
df.title = df.title.str.replace('\t|\n|\r|\f|\v', '', regex=True)
```

Testing out the output:


```python
df.loc[(df.filename == "AIZPND2.doc") | (df.filename == "JELGPPA2.pdf")]
```


```python
if len(df.loc[(df.rekviziti_email.str.contains(('\s'), na=False, regex=True)) | (df.kontaktpersona_mail.str.contains(('\s'), na=False, regex=True))]) == 0:
    print("Done")
```

### 3d. Wrong characters instead of "@"

Universal email regex: *"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$"*


```python
df.loc[~df.kontaktpersona_mail.str.contains("^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$", na=False)]
```

"(Ž", "Ģ", "(G)", "@)" instead of "@"


```python
for column in ["rekviziti_email", "kontaktpersona_mail"]:
    df[column] = df[column].str.replace('\(.', '@', regex=True) # "(G", "($" "(Ž"...
    df[column] = df[column].str.replace('@\)', '@', regex=True) # "@)"
```

"G", "Ģ", "GO", "OG", "Ž", "O" instead of "@"


```python
df.loc[df.rekviziti_email.str.contains('GO|OG|G|Ģ|Ž|O', na=False) | (df.kontaktpersona_mail.str.contains('GO|OG|G|Ģ|Ž|O', na=False))]
```


```python
for column in ["rekviziti_email", "kontaktpersona_mail"]:
    df[column] = df[column].str.replace('GO|OG', '@') 
```


```python
for column in ["rekviziti_email", "kontaktpersona_mail"]:
    df[column] = df[column].str.replace('G|Ģ|Ž|O', '@') 
```

Important: names containing one of these letters are OK, e.g. Didzis.Gaismins@*****.lv:


```python
df.kontaktpersona_mail.str.len().describe()
```

Mean email length is 58 chars (median is 24 chars), we will ignore messy (too long) email values for now.
To be on the safe side, we will use even lower value (48 chars) for the filter.


```python
# Didzis.Gaismins@*****.lv:

df.loc[((df.kontaktpersona_mail.str.len() < 48) & 
       ((df.kontaktpersona_mail.str.contains('.*?@.*?@.*?', na=False, regex=True)) |
       (df.kontaktpersona_mail.str.contains('@.*?@.*?', na=False, regex=True)))) | 
       ((df.rekviziti_email.str.len() < 48) & 
       ((df.rekviziti_email.str.contains('.*?@.*?@.*?', na=False, regex=True)) |
       (df.rekviziti_email.str.contains('@.*?@.*?', na=False, regex=True))))]
```

Manual data cleaning based on a visual assessment (faster to perform in this particular case):


```python
correction = {"CESUNP.doc": 'gints.garsa@*****.lv',
              "ADAND2.pdf": 'rita.ozolina@*****.lv',
              "CSDD.pdf": 'office@*****.lv',
              "LRVK.pdf": 'janis.gustins@*****.lv',
              "RFL.pdf": 'didzis.gaismins@*****.lv',
              "UGFA.pdf": 'margitagaile@*****.lv',
              "VK2.pdf": 'anita.ozola@*****.lv',
              "VSAA2.pdf": 'inese.cirule@*****.lv'}

for key, value in correction.items():
    df.loc[df.filename == key, 'kontaktpersona_mail'] = value
```

Testing out the output:


```python
df[df['filename'].isin(["CESUNP.doc", "ADAND2.pdf", "CSDD.pdf", "LRVK.pdf", 
                        "RFL.pdf", "UGFA.pdf", "VK2.pdf", "VSAA2.pdf"])]
```

### 3e. Too long / messy primary email values


```python
len(df.loc[(df.kontaktpersona_mail.str.len() >= 48)])
```

Looking for a Latvian email endings in this messy data (".lv" and possible OCR variations of ".lv"):


```python
df.loc[(df.kontaktpersona_mail.str.len() >= 48) & 
       (df.kontaktpersona_mail.str.contains(
           '.*?@*****.lv|.*?@.*?\,lv|.*?@.*?\|v|.*?@.*?_lv',
           na=False, regex=True))]
```


```python
df.loc[(df.filename == "ADNAMS.pdf") | 
       (df.filename == "VTPMAD.pdf") |
       (df.filename == "VSACR.pdf")] # test sample
```

Correcting OCR errors in ".lv" ending:


```python
df["kontaktpersona_mail"] = df["kontaktpersona_mail"].str.replace('.\!v|.\]v|\.ly|\.Iv|\.iv|\.lV|\,lv|\|v|_lv', '.lv', regex=True)
```

Testing out the output:


```python
df.loc[(df.filename == "ADNAMS.pdf") | 
       (df.filename == "VTPMAD.pdf") |
       (df.filename == "VSACR.pdf")]
```

Remove "Pasūtītāj..." and what comes after:


```python
df["kontaktpersona_mail"] = df["kontaktpersona_mail"].str.split("Pasūtītāj")
df["kontaktpersona_mail"] = df["kontaktpersona_mail"].str[0]

df.loc[(df.filename == "ADNAMS.pdf") | 
       (df.filename == "VTPMAD.pdf") |
       (df.filename == "VSACR.pdf")]
```


```python
len(df.loc[(df.kontaktpersona_mail.str.len() >= 48)])
```

Looking for a ".gov" emails:


```python
df.loc[(df.kontaktpersona_mail.str.len() >= 48) & (df.kontaktpersona_mail.str.contains('\.gov', na=False, regex=True))]
```

Quick manual data cleaning based on a visual assessment:


```python
df.loc[df.index == 492, 'kontaktpersona_mail'] = 'liaa@*****.lv'
df.loc[df.index == 1189, 'kontaktpersona_mail'] = 'vtua@*****.lv'
```

Testing out the output:


```python
df.loc[[492, 1189]]
```

Assigning what is left (long nonsense values) to Null:


```python
df.loc[df.kontaktpersona_mail.str.len() >= 48, 'kontaktpersona_mail'] = np.nan
if len(df.loc[(df.kontaktpersona_mail.str.len() >= 48)]) == 0:
    print("Done")
```

### 3f. Phone nr. before primary email 


```python
df.loc[df.kontaktpersona_mail.str.contains('Tālruņanumurs[0-9]+', na=False, regex=True)]
```


```python
df["kontaktpersona_mail"] = df["kontaktpersona_mail"].str.replace('Tālruņanumurs[0-9]+', '', regex=True)
if len(df.loc[df.kontaktpersona_mail.str.contains('Tālruņanumurs[0-9]+', na=False, regex=True)]) == 0:
    print("Done")
```

### 3g. Wrong contact names

Note: Latvian names may contain special characters: ĀČĒĢĪĶĻŅŠŪŽ (āčēģīķļņšūž).

Looking for a values that don't match name regex:


```python
warnings.filterwarnings("ignore", 'This pattern has match groups')

df.loc[~df.kontaktpersona_name.str.contains(
    '([A-ZĀČĒĢĪĶĻŅŠŪŽāčēģīķļņšūž]\w+(?=[\s\-][A-ZĀČĒĢĪĶĻŅŠŪŽāčēģīķļņšūž])(?:[\s\-][A-ZĀČĒĢĪĶĻŅŠŪŽāčēģīķļņšūž]\w+)+)',
    na=False, regex=True)]
```

Assigning these values to Null:


```python
df.loc[~df.kontaktpersona_name.str.contains(
    '([A-ZĀČĒĢĪĶĻŅŠŪŽāčēģīķļņšūž]\w+(?=[\s\-][A-ZĀČĒĢĪĶĻŅŠŪŽāčēģīķļņšūž])(?:[\s\-][A-ZĀČĒĢĪĶĻŅŠŪŽāčēģīķļņšūž]\w+)+)',
    na=False, regex=True), 'kontaktpersona_name'] = np.nan
```

### 3h. Empty email containing just ".lv"


```python
len(df.loc[df.kontaktpersona_mail == '.lv'])
```

Assigning these values to Null:


```python
df.loc[df.rekviziti_email == '.lv', "rekviziti_email"] = np.nan

df.loc[df.kontaktpersona_mail == '.lv', "kontaktpersona_mail"] = np.nan
if len(df.loc[df.kontaktpersona_mail == '.lv']) == 0:
    print("Done")
```

### 3i. Latvian characters in email


```python
len(df.loc[(df.rekviziti_email.str.contains('[ĀČĒĢĪĶĻŅŠŪŽāčēģīķļņšūž]', na=False, regex=True)) |
       (df.kontaktpersona_mail.str.contains('[ĀČĒĢĪĶĻŅŠŪŽāčēģīķļņšūž]', na=False, regex=True))])
```


```python
latvian_chars = ['Ā','Č','Ē','Ģ','Ī','Ķ','Ļ','Ņ','Š','Ū','Ž','ā','č','ē','ģ','ī','ķ','ļ','ņ','š','ū','ž']
english_chars = ['A','C','E','G','I','K','L','N','S','U','Z','a','c','e','g','i','k','l','n','s','u','z']

for char in latvian_chars:
    for column in ["rekviziti_email", "kontaktpersona_mail"]:
        df[column] = df[column].str.replace(char, english_chars[latvian_chars.index(char)]) 
```


```python
if len(df.loc[(df.rekviziti_email.str.contains('[ĀČĒĢĪĶĻŅŠŪŽāčēģīķļņšūž]', na=False, regex=True)) |
       (df.kontaktpersona_mail.str.contains('[ĀČĒĢĪĶĻŅŠŪŽāčēģīķļņšūž]', na=False, regex=True))]) == 0:
    print("Done")
```

### 3j. Dash and underscore characters in primary email 


```python
df.loc[df.kontaktpersona_mail.str.contains("^[\—\-\_][a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$", na=False, regex=True)]
```


```python
for column in ["rekviziti_email", "kontaktpersona_mail"]:
    df[column] = df[column].str.replace('[\—\-\_]', '', regex=True) 
    
if len(df.loc[df.kontaktpersona_mail.str.contains(
    "^[\—\-\_][a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$", na=False, regex=True)]) == 0:
    print("Done")
```

### 3k. Comma instead of a dot in email


```python
len(df.loc[(df.rekviziti_email.str.contains('\,', na=False, regex=True)) |
       (df.kontaktpersona_mail.str.contains('\,', na=False, regex=True))])
```


```python
for column in ["rekviziti_email", "kontaktpersona_mail"]:
    df[column] = df[column].str.replace('\,', '.', regex=True) 
```


```python
if len(df.loc[(df.rekviziti_email.str.contains('\,', na=False, regex=True)) |
       (df.kontaktpersona_mail.str.contains('\,', na=False, regex=True))]) == 0:
    print("Done")
```

Items that don't match email regex after all the manipulations:


```python
df.loc[~df.kontaktpersona_mail.str.contains(
    "^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$", na=False)]
```

Assigning these values to Null:


```python
# Items that don't match email regex after all the manipulations:
bad_email = list(df.loc[~df.kontaktpersona_mail.str.contains(
    "^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$", na=False)].index.values)

# Assigning these values to Null:
for index in bad_email:
    df.loc[df.index == index, 'kontaktpersona_mail'] = np.nan
```


```python
if len(df.loc[~df.kontaktpersona_mail.str.contains(
    r"(^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$)", na=False) & ~df.kontaktpersona_mail.isnull()]) == 0:
    print("Done")
```

### 3l. If there is no contact email, use general email when possible


```python
len(df.loc[df.kontaktpersona_mail.isnull()])
```


```python
df.loc[[29, 124]] # test sample
```


```python
df["kontaktpersona_mail"] = df["kontaktpersona_mail"].fillna(df["rekviziti_email"])
df.loc[df.kontaktpersona_mail.str.len() >= 47, 'kontaktpersona_mail'] = np.nan
```

Testing out the output:


```python
df.loc[[29, 124]]
```


```python
len(df.loc[df.kontaktpersona_mail.isnull()])
```

Drop these Null values, as they make no sense (both emails are NaN):


```python
df.dropna(subset=['kontaktpersona_mail'], inplace=True)
if len(df.loc[df.kontaktpersona_mail.isnull()]) == 0:
    print("Done")
```

### 3m. Remove duplicates


```python
df[df.duplicated(['kontaktpersona_mail'], keep=False)].sort_values(by=['kontaktpersona_mail'])
```

Drop few rows manually based on a visual assessment:


```python
df.drop([123, 492, 759], inplace=True)
```


```python
df.drop_duplicates(subset=['kontaktpersona_mail'], inplace=True)
if len(df[df.duplicated(['kontaktpersona_mail'], keep=False)].sort_values(by=['kontaktpersona_mail'])) == 0:
    print("Done")
```

---

### Data quality improvement checklist:

- [x] Null objects
- [x] Wrong file category (should be "Bad quality" if contact email is Null)
- [x] Unnecessary whitespace сharacters (e.g. "\t", "\n", "\r"...) 
- [x] Incorrect OCR results - wrong characters instead of "@" (e.g. "(Ž", "Ģ", "(G)"...)
- [x] Too long / messy email values (e.g. "CCA.al$ttkaeCatoja.byWa"IUaDskumentarekvizītus...")
- [x] Phone nr. before email (e.g. "Tālruņanumurs12345678Evija.Perkone@*****.lv")
- [x] Wrong contact names (e.g. "kas rīkojas", "parakstot šo", "(paraksts) mH!"...)
- [x] Empty email containing just ".lv"
- [x] Latvian characters in email (e.g. "saimniecība@*****.lv"...)
- [x] Dash and underscore characters in email (e.g. "—_ieva.libkena@*****.lv"...)
- [x] Comma instead of a dot in email
- [x] No contact email
- [x] Duplicates

## Step 4. Create a name greeting column.

Males names in Latvia end with -s, -š, -is, -us.<br>
Female names in Latvia end with -a, -e.

**When greeting males (for example in email), -s or -š should be omitted (e.g. "Jānis" - "Hello, Jāni!").**


```python
df["name_greeting"] = df["kontaktpersona_name"].str.split(" ")
df["name_greeting"] = df["name_greeting"].str[0]
df["name_greeting"].head()
```

Looking for a male names:


```python
print(len(df.loc[df.name_greeting.str.endswith('s', na=False)]))
print(len(df.loc[df.name_greeting.str.endswith('š', na=False)]))
print(len(df.loc[df.name_greeting.str.endswith('is', na=False)]))
print(len(df.loc[df.name_greeting.str.endswith('us', na=False)]))
```


```python
df['name_greeting'] = df['name_greeting'].str.rstrip('s')
df['name_greeting'] = df['name_greeting'].str.rstrip('š')

if (len(df.loc[df.name_greeting.str.endswith('s', na=False)]) == 0 & 
    len(df.loc[df.name_greeting.str.endswith('š', na=False)]) == 0 &
    len(df.loc[df.name_greeting.str.endswith('is', na=False)]) == 0):
    print("Done")
```

## Step 5. Save DataFrame to an Excel file.


```python
df.reset_index(drop=True, inplace=True)

df.to_excel('clean_data.xlsx', index=False)
df.to_csv('clean_data.csv', index=False)

df
```
