
### NOTE: Some code outputs are hidden here due to ethical and legal reasoning.

Personal data is subject to the protection requirements set out in The General Data Protection Regulation (GDPR).


```python
import os
import re
import sys
import time

from tika import parser
from wand.color import Color
from wand.image import Image
import cv2
import ghostscript
import pandas as pd
import PIL.Image
import pytesseract

# Tesseract parameters:
pytesseract.pytesseract.tesseract_cmd = 'C:\\Program Files (x86)\\Tesseract-OCR\\tesseract.exe'
TESSDATA_PREFIX = 'C:\\Program Files (x86)\\Tesseract-OCR\\tessdata'
```


```python
# Double check your Python runtime, and ensure the architectures match.

import struct
print(struct.calcsize("P") * 8)

from platform import python_version
print("Python version: " + python_version())

print("Ghostscript version: " + ghostscript.__version__)
```

    64
    Python version: 3.6.5
    Ghostscript version: 0.6
    


```python
def find_between(s, start, end):
    """Returns the string found between start string and end string (first occurrence)"""
    found_string = "N/A"
    found_string = s[s.find(start)+len(start):s.find(end)]
    return found_string
```

Only latvian domains (.lv) are suitable for client's needs, so gmail.com and such can be ignored.


```python
def check_for_email(block, email_title):
    """Returns an email string (latvian domains) found in the given text block"""
   
    email = None
    
    if '.!v' in block:
        email = find_between(block, email_title, '.!v')
    elif ']v' in block:
        email = find_between(block, email_title, '.]v')
    elif '.ly' in block:
        email = find_between(block, email_title, '.ly')
    elif '.Iv' in block:
        email = find_between(block, email_title, '.Iv')
    elif '.iv' in block:
        email = find_between(block, email_title, '.iv')
        
    # keep ".lv" last as the most correct one:
    elif '.lv' in block:
        email = find_between(block, email_title, '.lv') 
        
    if email != None:    
        email = email[2:] + '.lv'
    
    
    return email
```


```python
def get_contacts_from_string(pdf_string):
    """Returns a list of strings found in the main text"""
    
    #####

    # HIDDEN CODE DUE TO CONFIDENTIAL BUSINESS INFORMATION OR OTHER RESTRICTIONS. THE CODE IS AVAILABLE ON REQUEST.     

    #####

    return [title, rekviziti_email, kontaktpersona_name, kontaktpersona_mail]
```


```python
folder = 'extracted' # name of the folder containing PDF files to work with
```

**Step 1. Create a DataFrame from “good” files.**


```python
df_list = []
scanned = []

for document in os.listdir(folder):

    pdf_string = ""
    title = ""
    rekviziti_email = ""
    kontaktpersona_name = ""
    kontaktpersona_mail = ""
    file_category = ""
    
 
    try:
        raw = parser.from_file(os.getcwd() + "\\" + folder + "\\" + document)
        pdf_string = raw['content']

        # WE ARE WORKING WITH A "GOOD" / READABLE FILE (PDF OR MS WORD):

        file_category = "Good"

        title, rekviziti_email, kontaktpersona_name, kontaktpersona_mail = get_contacts_from_string(pdf_string)

        # To avoif getting values like "1605[3].pdf" from PDF document title:
        if ".pdf" not in title:
        
            df_list.append({'title': title,
                            'rekviziti_email': rekviziti_email,
                            'kontaktpersona_name': kontaktpersona_name,
                            'kontaktpersona_mail': kontaktpersona_mail,
                            'file_category': file_category,
                            'filename': document})
        
        s = 'SUCCESS: ' + document
        
    except: 
        
        scanned.append(document)
        s = 'ERROR EXTRACTING CONTENT: ' + document
        
        # ERROR EXTRACTING CONTENT FROM PARSER. WE ARE PROBABLY DEALING WITH A SCANNED FILE.
    
    print(s + " "*100, end="\r") # erase output and print on the same line
    
    df_good = pd.DataFrame(df_list, columns = ['filename', 'title', 'rekviziti_email', 'kontaktpersona_name', 'kontaktpersona_mail', 'file_category'])
```

    SUCCESS: D32PII.pdf                                                                                                                       

    2019-01-24 12:36:14,304 [MainThread  ] [WARNI]  Tika server returned status: 422
    

    ERROR EXTRACTING CONTENT: ZVS.pdf                                                                                                                        


```python
df_good
```

**Step 2. Create PNG images from scanned files.**
<br>Files with error are to be inspected manually.

- Requires installing https://imagemagick.org/download/binaries/
- Requires installing https://www.ghostscript.com/download/gsdnld.html


```python
good_scanned = []
bad_scanned = []

for document in scanned:
    
    try: # Convert scanned PDF to image:
        
        img_filename = os.getcwd() + "\\png\\" + os.path.splitext(document)[0] + ".png"
            
        with Image(filename=(os.getcwd() + "\\" + folder + "\\" + document), resolution=300) as img:

            with Image(width=img.width, height=img.height, background=Color("white")) as bg:
                bg.composite(img,0,0)
                bg.save(filename=img_filename)  
        
        # Reduce the noise with median blurring to improve the quality of the output:
        img = cv2.imread(img_filename, 0)
        img = cv2.medianBlur(img, 3)
        cv2.imwrite(img_filename, img)
    
        good_scanned.append(document)

    
    except:
        
        bad_scanned.append(document)

        
print('SUCCESS:')
print(good_scanned)

print('\nERROR SAVING IMAGE:')
print(bad_scanned)
```

    SUCCESS:
    ['9VSK.pdf', 'ADAND.pdf', 'ADAND2.pdf', 'ADASL.pdf', 'ADNAMS.pdf', 'ADU.pdf', 'AGLND.pdf', 'AGLND2.pdf', 'AIAVS.pdf', 'AIC.pdf', 'AIM.pdf', 'AIM2.pdf', 'AIP.pdf', 'AIZKPVS.pdf', 'AIZKPVS2.pdf', 'AIZKR.pdf', 'AIZPN.pdf', 'AIZPND.pdf', 'AKADP.pdf', 'AKNNP.pdf', 'AKPNS.pdf', 'AKPNS2.pdf', 'ALAAS.pdf', 'ALBATR.pdf', 'ALOND.pdf', 'ALOND2.pdf', 'ALSL.pdf', 'ALSND.pdf', 'ALSPS.pdf', 'ALSPS2.pdf', 'ALTUM.pdf', 'ALUNAMI.pdf', 'ALUNP.pdf', 'ALUPSS.pdf', 'AM.pdf', 'AM2.pdf', 'AMANP.pdf', 'AMISK.pdf', 'AMOIC.pdf', 'AMOIC2.pdf', 'APKAUDZ.pdf', 'APV.pdf', 'APV2.pdf', 'APVJRPP.pdf', 'ARKUK.pdf', 'AT.pdf', 'ATD.pdf', 'ATD2.pdf', 'AVG.pdf', 'AVSAC.pdf', 'BA.pdf', 'BA2.pdf', 'BABNP.pdf', 'BABNP2.pdf', 'BALDD.pdf', 'BALDD2.pdf', 'BARAV.pdf', 'BARAV2.pdf', 'BASEINS.pdf', 'BAU2VS.pdf', 'BAU2VS2.pdf', 'BAUKC.pdf', 'BAUMS.pdf', 'BAUNMM.pdf', 'BAUPPS.pdf', 'BAUSKSS.pdf', 'BAUSL.pdf', 'BAUSL2.pdf', 'BAUVG.pdf', 'BBJC.pdf', 'BELAVPP.pdf', 'BEREZ.pdf', 'BEVERNP.pdf', 'BEVERNP2.pdf', 'BGSA.pdf', 'BIKUR.pdf', 'BIOR.pdf', 'BIOR2.pdf', 'BJBSR.pdf', 'BJCRSP.pdf', 'BJSSR.pdf', 'BKS.pdf', 'BKUS.pdf', 'BKUS2.pdf', 'BNADM.pdf', 'BNADM2.pdf', 'BNLAUB.pdf', 'BNSD.pdf', 'BPPIIP.pdf', 'BPPIIZ.pdf', 'BPPIIZ2.pdf', 'BROCNP.pdf', 'BRUNPP.pdf', 'BTI.pdf', 'BULDVS.pdf', 'BULDVS2.pdf', 'BURTNP.pdf', 'BVKB.pdf', 'CAA.pdf', 'CARNKS.pdf', 'CARNKS2.pdf', 'CBK.pdf', 'CERPP.pdf', 'CESUK.pdf', 'CESUK2.pdf', 'CESUNSD.pdf', 'CESUPVS.pdf', 'CFI.pdf', 'CFI2.pdf', 'CFLA.pdf', 'CFLA2.pdf', 'CIBLANP.pdf', 'CIECIPS.pdf', 'CIELAV.pdf', 'CIEMATS.pdf', 'CIPSK.pdf', 'CND.pdf', 'CND2.pdf', 'CODPS.pdf', 'CSDD.pdf', 'CSI.pdf', 'CSP.pdf', 'CSP2.pdf', 'D12VSK.pdf', 'D13PII.pdf', 'D15SPII.pdf', 'DAILEST.pdf', 'DAP.pdf', 'DAUDZPP.pdf', 'DAUG3VS.pdf', 'DAUGMK.pdf', 'DAUGNKP.pdf', 'DAUGPPP.pdf', 'DAUGPSP.pdf', 'DAUGVT.pdf', 'DAUGVT2.pdf', 'DAUKSPP.pdf', 'DAUND.pdf', 'DAUND2.pdf', 'DAUPD.pdf', 'DAUPD2.pdf', 'DAUPKP.pdf', 'DAUSILT.pdf', 'DAVPP.pdf', 'DAVS.pdf', 'DBJCJ.pdf', 'DCPVS.pdf', 'DCPVS2.pdf', 'DEMPP.pdf', 'DERPELE.pdf', 'dkp.pdf', 'DLKC.pdf', 'DMRMC.pdf', 'DMV.pdf', 'DNCB.pdf', 'DNDSC.pdf', 'DNMM.pdf', 'DNSD.pdf', 'DOBNP.pdf', 'DOBSL.pdf', 'DOBSL2.pdf', 'DOBUD.pdf', 'doc01777720141119162720.pdf', 'DPDSLP.pdf', 'DPNS.pdf', 'DPNS2.pdf', 'DPSATC.pdf', 'DRS.pdf', 'DRS2.pdf', 'DS.pdf', 'DTS.pdf', 'DU.pdf', 'DUNDNP.pdf', 'DUNDVC.pdf', 'DURBND.pdf', 'DUSIA.pdf', 'DVG.pdf', 'DVI.pdf', 'DVI2.pdf', 'DZIVKS.pdf', 'DZKSU.pdf', 'EBUSI.pdf', 'EDI.pdf', 'EJIHL.pdf', 'EM.pdf', 'EM2.pdf', 'EMMVS.pdf', 'ENGND.pdf', 'ENGND2.pdf', 'ENGPP.pdf', 'ERGAVS.pdf', 'ERGNP.pdf', 'ERGNP2.pdf', 'ESD.pdf', 'ESD2.pdf', 'ESTETS.pdf', 'EZPP.pdf', 'FEI.pdf', 'FISKDP.pdf', 'FKTK.pdf', 'FM.pdf', 'FSI.pdf', 'GAILPP.pdf', 'GALGPP.pdf', 'GARKND.pdf', 'GAUJIPS.pdf', 'GHES.pdf', 'GIBPP.pdf', 'GIIC.pdf', 'GINTE.pdf', 'GNN.pdf', 'GROBND.pdf', 'GROBSC.pdf', 'GTKMC.pdf', 'GULBPP.pdf', 'GULNJCB.pdf', 'IAUI.pdf', 'IECND.pdf', 'IECND2.pdf', 'IECVSK.pdf', 'IEM.pdf', 'IEMIC.pdf', 'IEMIC2.pdf', 'IEMNVA.pdf', 'IEMPOL.pdf', 'IEMPOL2.pdf', 'IEMVSC.pdf', 'IEVP.pdf', 'IEVP2.pdf', 'IKSM.pdf', 'IKSNP.pdf', 'IKSNP2.pdf', 'IKVD.pdf', 'ILSVS.pdf', 'INCUKND.pdf', 'INDRPP.pdf', 'INP.pdf', 'IRLAVPP.pdf', 'ISKS.pdf', 'ISLVS .pdf', 'ISS.pdf', 'IUB.pdf', 'IUB2.pdf', 'IZM.pdf', 'IZM2.pdf', 'JANMPVS.pdf', 'JAUND.pdf', 'JAUND2.pdf', 'JAUNGPP.pdf', 'JBSAC.pdf', 'JEKPP.pdf', 'JEKPP2.pdf', 'JELGAVS.pdf', 'JELGAVS2.pdf', 'JELGNP.pdf', 'JELGPBT.pdf', 'JELGPOL.pdf', 'JELGPOL2.pdf', 'JELGPPA.pdf', 'JELGPPP.pdf', 'JELGRTC.pdf', 'JELGSLP.pdf', 'JELPD.pdf', 'JELPD2.pdf', 'JELPPIK.pdf', 'JELPS.pdf', 'JELPS2.pdf', 'JGAVS.pdf', 'JIP.pdf', 'JIRMV.pdf', 'JKNP.pdf', 'JKP.pdf', 'JMMSK.pdf', 'JMRMV.pdf', 'JMRMV2.pdf', 'JMV.pdf', 'JMV2.pdf', 'JN.pdf', 'JNIP.pdf', 'JPA.pdf', 'JPA2.pdf', 'JRCS.pdf', 'JRCS2.pdf', 'JRRMV.pdf', 'JRRMV2.pdf', 'JRT.pdf', 'JS.pdf', 'JS2.pdf', 'JSPA.pdf', 'JSPA2.pdf', 'JSSC.pdf', 'JURKANT.pdf', 'JURMG.pdf', 'JURMU.pdf', 'JURPD.pdf', 'JURPD2.pdf', 'JURPDLP.pdf', 'JURPDLP2.pdf', 'JVLMA.pdf', 'JZB.pdf', 'KALNPP.pdf', 'KALSNK.pdf', 'KAND.pdf', 'KANDNKP.pdf', 'KARSLMN.pdf', 'KARSNP.pdf', 'KARSNP2.pdf', 'KAZDPP.pdf', 'KEGND.pdf', 'KEKNP.pdf', 'KEKVS.pdf', 'KEKVS2.pdf', 'KF.pdf', 'KIS.pdf', 'KIVS.pdf', 'KKI.pdf', 'KKMVS.pdf', 'KKP.pdf', 'KM.pdf', 'KM2.pdf', 'KNAB.pdf', 'KNAB2.pdf', 'KNMC.pdf', 'KOCND.pdf', 'KOCND2.pdf', 'KOKND.pdf', 'KOKND2.pdf', 'KONKP.pdf', 'KONKP2.pdf', 'KRASL.pdf', 'KRASL2.pdf', 'KRASLPS.pdf', 'KRASLVG.pdf', 'KRAVVS.pdf', 'KRIMND.pdf', 'KRIMND2.pdf', 'KRNPIIP.pdf', 'KRPIIP.pdf', 'KRSS.pdf', 'KULDS.pdf', 'KULNP.pdf', 'KULNP2.pdf', 'KURPLR.pdf', 'KURPLR2.pdf', 'KURSPP.pdf', 'KVC.pdf', 'LAD.pdf', 'LAD2.pdf', 'LAFI.pdf', 'LAPMPP.pdf', 'LATGPR.pdf', 'LATGRAA.pdf', 'LATJA.pdf', 'LATJA2.pdf', 'LATVEN.pdf', 'LAU.pdf', 'LAUDC.pdf', 'LAUDC2.pdf', 'LB.pdf', 'LB2.pdf', 'LBF.pdf', 'LCB.pdf', 'LDM.pdf', 'LDMV.pdf', 'LDMV2.pdf', 'LDZ.pdf', 'LDZKS.pdf', 'LEBM.pdf', 'LEJCPP.pdf', 'LGA.pdf', 'LGA2.pdf', 'LGIA.pdf', 'LGIA2.pdf', 'LHEI.pdf', 'LI.pdf', 'LIAA.pdf', 'LIAA2.pdf', 'LIBAGPP.pdf', 'LIC.pdf', 'LIELVNP.pdf', 'LIELVNP2.pdf', 'LIEPAPP.pdf', 'LIEPPP.pdf', 'LIEPU.pdf', 'LIEPU2.pdf', 'LIEPVK.pdf', 'LIGN.pdf', 'LIGND.pdf', 'LIMNP.pdf', 'LIMNP2.pdf', 'LIP.pdf', 'LIVND.pdf', 'LIVND2.pdf', 'LIVSIL.pdf', 'LIVSL.pdf', 'LIZPP.pdf', 'LI_2.pdf', 'LJA.pdf', 'LJA2.pdf', 'LJK.pdf', 'LK.pdf', 'LK2.pdf', 'LKA.pdf', 'LKA2.pdf', 'LKK.pdf', 'LLKIC.pdf', 'LLKIC2.pdf', 'LLT.pdf', 'LLU.pdf', 'LLU2.pdf', 'LM.pdf', 'LM2.pdf', 'LMA.pdf', 'LNA.pdf', 'LNA2.pdf', 'LNARH.pdf', 'LNARH2.pdf', 'LNB.pdf', 'LNB2.pdf', 'LNMC.pdf', 'LNMM.pdf', 'LNMM2.pdf', 'LNS.pdf', 'LNS2.pdf', 'LNSO.pdf', 'LNVM.pdf', 'LNVM2.pdf', 'LOK.pdf', 'LOV.pdf', 'LPESPS.pdf', 'LPESPS2.pdf', 'LPIPCS.pdf', 'LPMUMS.pdf', 'LPV.pdf', 'LPV2.pdf', 'LREMTE.pdf', 'LRP.pdf', 'LRP2.pdf', 'LRPV.pdf', 'LRPV2.pdf', 'LRS.pdf', 'LRS2.pdf', 'LRVK.pdf', 'LRVK2.pdf', 'LSFP.pdf', 'LSO.pdf', 'LTV.pdf', 'LTV2.pdf', 'LTZI.pdf', 'LU.pdf', 'LU2.pdf', 'LUBI.pdf', 'LUBNP.pdf', 'LUBNP2.pdf', 'LUDSL.pdf', 'LUDSL2.pdf', 'LUDZNP.pdf', 'LUFI.pdf', 'LULFMI.pdf', 'LUMII.pdf', 'LUMII2.pdf', 'LURMK.pdf', 'LUZPV.pdf', 'LV.pdf', 'LV1G.pdf', 'LVAEI2.pdf', 'LVAFA.pdf', 'LVAI.pdf', 'LVC.pdf', 'LVC2.pdf', 'LVGMC.pdf', 'LVGMC2.pdf', 'LVM.pdf', 'LVM2.pdf', 'LVPK.pdf', 'LVPK2.pdf', 'LVRTC.pdf', 'LVRTC2.pdf', 'LVT.pdf', 'LVT2.pdf', 'LZA.pdf', 'MADLPP.pdf', 'MADNP.pdf', 'MADSL.pdf', 'MADSL2.pdf', 'MALIPS.pdf', 'MALK.pdf', 'MALK2.pdf', 'MALNK.pdf', 'MALNK2.pdf', 'MALPND.pdf', 'MALPND2.pdf', 'MALPPVS.pdf', 'MARKP.pdf', 'MARUPND.pdf', 'MARUPND2.pdf', 'MAZSNP.pdf', 'MAZSNP2.pdf', 'MEDPP.pdf', 'MELPR.pdf', 'MERSNP.pdf', 'MEZGPS.pdf', 'MEZPS.pdf', 'MGILDE.pdf', 'MILPO.pdf', 'MMA.pdf', 'MNA.pdf', 'MNA2.pdf', 'MPP.pdf', 'MPS.pdf', 'MSG.pdf', 'MVM.pdf', 'MVPS.pdf', 'NAMZIN.pdf', 'NATOIC.pdf', 'NAUJPP.pdf', 'NAUJPP2.pdf', 'NBD.pdf', 'NBD2.pdf', 'NBSAS.pdf', 'NBSNP.pdf', 'NBSNP2.pdf', 'NERNP.pdf', 'NERNP2.pdf', 'NIGRPP.pdf', 'NKC.pdf', 'NKC2.pdf', 'NKI.pdf', 'NMPD.pdf', 'NMPD2.pdf', 'NMVSK.pdf', 'NOVPP.pdf', 'NPS.pdf', 'NRCV.pdf', 'NRCV2.pdf', 'NSKII.pdf', 'NSKII2.pdf', 'NVA.pdf', 'NVA2.pdf', 'OAVS.pdf', 'OBJC.pdf', 'OCREZ.pdf', 'OCVENTS.pdf', 'OGRBASK.pdf', 'OGRERS.pdf', 'OGRERS2.pdf', 'OGRNAM.pdf', 'OGRNKC.pdf', 'OGRNP.pdf', 'OKSDU.pdf', 'OLAINP.pdf', 'OLUS.pdf', 'OMTK.pdf', 'OPERA.pdf', 'OPERA2.pdf', 'OPVS.pdf', 'OSI.pdf', 'OTRNC.pdf', 'OTRNC2.pdf', 'OVPIIZS.pdf', 'OVT.pdf', 'OVT2.pdf', 'OZOLND.pdf', 'OZOLND2.pdf', 'PA.pdf', 'PA2.pdf', 'PAMSIPS.pdf', 'PARGNP.pdf', 'PAVILNP.pdf', 'PCABC.pdf', 'PCABC2.pdf', 'PIEDRPP.pdf', 'PIEJURA.pdf', 'PIIKURZ.pdf', 'PIKSP.pdf', 'PIRNC.pdf', 'PIRNC2.pdf', 'PJS.pdf', 'PJSPP.pdf', 'PKC.pdf', 'PKV.pdf', 'PLND.pdf', 'PMI.pdf', 'PMLP.pdf', 'pmlp2.pdf', 'PMMS.pdf', 'PPIIKAM.pdf', 'PPIIZEM.pdf', 'PPP.pdf', 'PRAUDA.pdf', 'PREIAVS.pdf', 'PRESL.pdf', 'PRESL2.pdf', 'PRIEDES.pdf', 'PRIEKNP2.pdf', 'PRIEKSL.pdf', 'PRIENP.pdf', 'PRIESEL.pdf', 'PRVS.pdf', 'PSK.pdf', 'PTAC.pdf', 'PTAC2.pdf', 'PV.pdf', 'PV2.pdf', 'PVACZ.pdf', 'PVD.pdf', 'R110PII.pdf', 'R123PII.pdf', 'R125PII.pdf', 'R139PII.pdf', 'R141PII.pdf', 'R145PII.pdf', 'R148PII.pdf', 'R149SAU.pdf', 'R14PII.pdf', 'R14VMVS.pdf', 'R154PII.pdf', 'R160PII.pdf', 'R160PII2.pdf', 'R162PII.pdf', 'R167PII.pdf', 'R169PII.pdf', 'R170PII.pdf', 'R172PII.pdf', 'R182PII.pdf', 'R18VMVS.pdf', 'R192PII.pdf', 'R193PII.pdf', 'R197PII.pdf', 'R200PII.pdf', 'R209PII.pdf', 'R213PII.pdf', 'R21PII.pdf', 'R221PII.pdf', 'R221PII_2.pdf', 'R224PII.pdf', 'R225PII.pdf', 'R233PII.pdf', 'R234PII.pdf', 'R236PII.pdf', 'R239PII.pdf', 'R244PII.pdf', 'R255PII.pdf', 'R255PII2.pdf', 'R258PII.pdf', 'R259PII.pdf', 'R261PII.pdf', 'R264PII.pdf', 'R267PII.pdf', 'R270PII.pdf', 'R272PII.pdf', 'R2SIPS.pdf', 'R31VS.pdf', 'R3BJSS.pdf', 'R3G.pdf', 'R40PII.pdf', 'R41PII.pdf', 'R46PII.pdf', 'R49PII.pdf', 'R49PII2.pdf', 'R49VS.pdf', 'R4PIIA.pdf', 'R5PIIC.pdf', 'R5SIPS.pdf', 'R61PII.pdf', 'R62PII.pdf', 'R68PII.pdf', 'R80PII.pdf', 'R81PII.pdf', 'R88PII.pdf', 'R8PII.pdf', 'RADIO.pdf', 'RADIO2.pdf', 'RAID.pdf', 'RAILNET.pdf', 'RAKUS.pdf', 'RAKUS2.pdf', 'RANKPP.pdf', 'RANKPVS.pdf', 'RAUNA.pdf', 'RAV.pdf', 'RAZNA.pdf', 'RBFLOTE.pdf', 'RBRAIL.pdf', 'RBT.pdf', 'RBVSK.pdf', 'RCB.pdf', 'RCK.pdf', 'RCK2.pdf', 'RCT.pdf', 'RDFD.pdf', 'RDFD2.pdf', 'RDID.pdf', 'RDID2.pdf', 'RDITC.pdf', 'RDKS.pdf', 'RDKS2.pdf', 'RDLD.pdf', 'RDLD2.pdf', 'RDMV.pdf', 'RDMV2.pdf', 'RDMVD.pdf', 'RDPAD.pdf', 'RDPAD2.pdf', 'RDPIKN.pdf', 'RDPIP.pdf', 'RDSD.pdf', 'RDSD2.pdf', 'RDSP.pdf', 'RDZN.pdf', 'RDZPII.pdf', 'REA.pdf', 'REAVS.pdf', 'REZNAMI.pdf', 'REZNP.pdf', 'REZPD.pdf', 'REZSL.pdf', 'REZSL2.pdf', 'REZVAC.pdf', 'RFL.pdf', 'RFSK.pdf', 'RG.pdf', 'RGPS.pdf', 'RHV.pdf', 'RIAID.pdf', 'RIEBND.pdf', 'RIEBND2.pdf', 'RIG2014.pdf', 'RIGASN.pdf', 'RIGAZOO.pdf', 'RIPSL.pdf', 'RIPSL2.pdf', 'RISERV.pdf', 'RIVBDZT.pdf', 'RJC.pdf', 'RJC2.pdf', 'RKG.pdf', 'RLIPS.pdf', 'RMDVS.pdf', 'RMEZI.pdf', 'RMIVS.pdf', 'RMM.pdf', 'RMM2.pdf', 'RNP.pdf', 'RNP2.pdf', 'RNPIIM.pdf', 'ROBPP.pdf', 'ROJA.pdf', 'ROP.pdf', 'ROPNP.pdf', 'ROPNP2.pdf', 'ROTSL.pdf', 'ROTSL2.pdf', 'RP.pdf', 'RPB.pdf', 'RPBJC.pdf', 'RPDID.pdf', 'RPIIAB.pdf', 'RPIIASN.pdf', 'RPIIAUS.pdf', 'RPIIDOM.pdf', 'RPIIDZ.pdf', 'RPIIKAM.pdf', 'RPIIMAR.pdf', 'RPIIMEZ.pdf', 'RPIIPAS.pdf', 'RPIIPIL.pdf', 'RPIIPR.pdf', 'RPIIPUC.pdf', 'RPIIR.pdf', 'RPIIROT.pdf', 'RPIISAP.pdf', 'RPIISPR.pdf', 'RPIIZ.pdf', 'RPIIZIL.pdf', 'RPIIZVA.pdf', 'RPIVA.pdf', 'RPKIS.pdf', 'RPMK.pdf', 'RPMK2.pdf', 'RPNC.pdf', 'RPP.pdf', 'RPP2.pdf', 'RPPVS.pdf', 'RPR.pdf', 'RPR2.pdf', 'RPRVS.pdf', 'RPRVS2.pdf', 'RRSL.pdf', 'RSACG.pdf', 'RSACM.pdf', 'RSACM2.pdf', 'RSMPV.pdf', 'RSMPV2.pdf', 'RSOCD.pdf', 'RSPIPS.pdf', 'RSPSK.pdf', 'RSU.pdf', 'rsu2.pdf', 'RSUSI.pdf', 'RTK.pdf', 'RTRAS.pdf', 'RTRAS2.pdf', 'RTT.pdf', 'RTU.pdf', 'RUCND.pdf', 'RUGND.pdf', 'RUJNP.pdf', 'RUNDALE.pdf', 'RUNDALE2.pdf', 'RUNDM.pdf', 'RVESC.pdf', 'RVKM.pdf', 'RVT.pdf', 'RVT2.pdf', 'RZID.pdf', 'RZOLPS.pdf', 'RZPSK.pdf', 'SACAGL.pdf', 'SACTER.pdf', 'SAEIMA.pdf', 'SAEIMA2.pdf', 'SAL1VS.pdf', 'SAL2VS.pdf', 'SALASND.pdf', 'SALASND2.pdf', 'SALDNAP.pdf', 'SALKD.pdf', 'SALND.pdf', 'SALND2.pdf', 'SALNP.pdf', 'SALTAV.pdf', 'SALVC.pdf', 'SAM.pdf', 'SAM2.pdf', 'SAMC.pdf', 'SAMC2.pdf', 'SAMCSIA.pdf', 'SAMPN.pdf', 'SAMPN2.pdf', 'SANTEX.pdf', 'SATVT.pdf', 'SAUL.pdf', 'SAUL2.pdf', 'SAULSL.pdf', 'SAULSL2.pdf', 'SAULVSK.pdf', 'SERNIK.pdf', 'SIALA.pdf', 'SIF.pdf', 'SIGAA.pdf', 'SIGNBIB.pdf', 'SIGND.pdf', 'SIGPV.pdf', 'SILAVA.pdf', 'SIVA.pdf', 'SIVA2.pdf', 'SKAVS.pdf', 'SKAVS2.pdf', 'SKILSAM.pdf', 'SKPP.pdf', 'SKRIND.pdf', 'SKRUDPP.pdf', 'SMILT.pdf', 'SMILT2.pdf', 'SMILTND.pdf', 'SMIVS.pdf', 'SMVA.pdf', 'SND.pdf', 'SNP.pdf', 'SPAAO.pdf', 'SPIIJAN.pdf', 'SPIIS.pdf', 'SPKC.pdf', 'SPKC2.pdf', 'SPNS.pdf', 'SPP.pdf', 'SPV.pdf', 'SPVS.pdf', 'SPVS2.pdf', 'SSARK.pdf', 'STAMPP.pdf', 'STELM.pdf', 'STELM2.pdf', 'STOPND.pdf', 'STOPND2.pdf', 'STOPPA.pdf', 'STRND.pdf', 'STRPP.pdf', 'STRSL.pdf', 'STRSL2.pdf', 'STSELI.pdf', 'SUNTPP.pdf', 'SZA.pdf', 'SZPP.pdf', 'TA.pdf', 'TA2.pdf', 'TALSND.pdf', 'TALSND2.pdf', 'TALSNIP.pdf', 'TALSU.pdf', 'TALSUN.pdf', 'TAURPP.pdf', 'TAVA.pdf', 'TCL.pdf', 'TIRZPP.pdf', 'TLH.pdf', 'TM.pdf', 'TM2.pdf', 'TNA.pdf', 'TNA2.pdf', 'TOS.pdf', 'TRD.pdf', 'TRRNC.pdf', 'TRRNC2.pdf', 'TSB.pdf', 'TSB2.pdf', 'TUDEPP.pdf', 'TUKND.pdf', 'TUKND2.pdf', 'TUKSL.pdf', 'TUKSL2.pdf', 'TUMVSK.pdf', 'TURAIDA.pdf', 'TURLAVA.pdf', 'UDEK.pdf', 'UGFA.pdf', 'UGFA2.pdf', 'UR.pdf', 'URBIN.pdf', 'UZVSK.pdf', 'VAAD.pdf', 'VAAD2.pdf', 'VADC.pdf', 'VADC2.pdf', 'VADPP.pdf', 'VAINND.pdf', 'VALAPP.pdf', 'VALKA.pdf', 'VALKM.pdf', 'VALMIER.pdf', 'VALMIER2.pdf', 'VALMN.pdf', 'VALNB.pdf', 'VALODA.pdf', 'VALODA2.pdf', 'VARAM.pdf', 'VARAM2.pdf', 'VARKAVA.pdf', 'VARNP.pdf', 'VARNP2.pdf', 'VASKOLA.pdf', 'VASKOLA2.pdf', 'VBP.pdf', 'VCIL.pdf', 'VCKENG.pdf', 'VDA.pdf', 'VDC.pdf', 'VDEAVK.pdf', 'VDEAVK2.pdf', 'VDI.pdf', 'VDI2.pdf', 'VDZTI.pdf', 'VEC.pdf', 'VECND.pdf', 'VECND2.pdf', 'VECSAPP.pdf', 'VECSPP.pdf', 'VECSSK.pdf', 'VECVS.pdf', 'VENMV.pdf', 'VENPDIP.pdf', 'VENPDSP.pdf', 'VENPOL.pdf', 'VENSANS.pdf', 'VENTA.pdf', 'VENTA2.pdf', 'VENTEH2.pdf', 'VENTM.pdf', 'VENTNI.pdf', 'VENTPD.pdf', 'VENTS.pdf', 'VENTSNP.pdf', 'VI.pdf', 'VI2.pdf', 'VIAA.pdf', 'VIAA2.pdf', 'VIAVS.pdf', 'VID.pdf', 'VID2.pdf', 'VIDA.pdf', 'VIDKONC.pdf', 'VIDZPR.pdf', 'VIDZS.pdf', 'VIDZS2.pdf', 'VIESNP.pdf', 'VIESNP2.pdf', 'VIGANTS.pdf', 'VILND.pdf', 'VILNP.pdf', 'VIPVS.pdf', 'VISC.pdf', 'VISC2.pdf', 'VISKI.pdf', 'VISKPP.pdf', 'VK.pdf', 'VK2.pdf', 'VKALT.pdf', 'VKALT2.pdf', 'VKANC.pdf', 'VKANC2.pdf', 'VKKF.pdf', 'VKKF2.pdf', 'VKPAI.pdf', 'VKPAI2.pdf', 'VLABK.pdf', 'VLMSPR.pdf', 'VM.pdf', 'VM2.pdf', 'VMD.pdf', 'VMD2.pdf', 'VMESF.pdf', 'VMESF2.pdf', 'VMNVD.pdf', 'VMNVD2.pdf', 'VNC.pdf', 'VNI.pdf', 'VNI2.pdf', 'VP.pdf', 'VP2.pdf', 'VPD.pdf', 'VPD2.pdf', 'VPPP.pdf', 'VPRLT.pdf', 'VPRLT2.pdf', 'VPRRP.pdf', 'VPRVS.pdf', 'VPV.pdf', 'VPV2.pdf', 'VPVB.pdf', 'VPVSK.pdf', 'VPVSK2.pdf', 'VRAA.pdf', 'VRAA2.pdf', 'VRDP.pdf', 'VRK.pdf', 'VRK2.pdf', 'VRSAP.pdf', 'VRSGP.pdf', 'VRSGP2.pdf', 'VRSLUP.pdf', 'VRSRP.pdf', 'VRSVILP.pdf', 'VRVP.pdf', 'VSAA.pdf', 'VSAA2.pdf', 'VSACK.pdf', 'VSACK2.pdf', 'VSACL.pdf', 'VSACL2.pdf', 'VSACR.pdf', 'VSACR2.pdf', 'VSACV.pdf', 'VSACV2.pdf', 'VSACZ.pdf', 'VSPCD.pdf', 'VTEB.pdf', 'VTEB2.pdf', 'VTMEC.pdf', 'VTMEC2.pdf', 'VTPMAD.pdf', 'VTPMAD2.pdf', 'VTUA.pdf', 'VUGD.pdf', 'VUGD2.pdf', 'VVC.pdf', 'VVC2.pdf', 'VVD.pdf', 'VVD2.pdf', 'VZD.pdf', 'ZAAO.pdf', 'ZEMGVS.pdf', 'ZEMPR.pdf', 'ZILKAA.pdf', 'ZILLTD.pdf', 'ZILUPE.pdf', 'ZIRPP.pdf', 'ZKRS.pdf', 'ZKRS2.pdf', 'ZM.pdf', 'ZM2.pdf', 'ZMNI.pdf', 'ZRKAC.pdf', 'ZSS.pdf', 'ZSS2.pdf', 'ZVA.pdf', 'ZVA2.pdf', 'ZVS.pdf']
    
    ERROR SAVING IMAGE:
    ['AMASK.docx', 'D4SPII.pdf']
    

**Step 3. Extract text from the images & create a DataFrame from scanned files.**

- Requires installing https://github.com/tesseract-ocr/tesseract/wiki


```python
df_list = []

for document in good_scanned:
    
    img_filename = os.getcwd() + "\\png\\" + os.path.splitext(document)[0] + ".png"

    file_category = "Scanned"

    # Extract text from the image:
    pdf_string = pytesseract.image_to_string(PIL.Image.open(img_filename).convert("RGB"), 
                                             lang='lav+eng') # "lav" for latvian
                                                             # "eng" for multilingual characters like "@" 

    title, rekviziti_email, kontaktpersona_name, kontaktpersona_mail = get_contacts_from_string(pdf_string)

    # If there is no "at sign" in the email address, we are probably dealing with a bad quality scan:
    if kontaktpersona_mail != None and "@" not in kontaktpersona_mail:
        file_category = "Bad quality"
    
    df_list.append({'title': title,
                    'rekviziti_email': rekviziti_email,
                    'kontaktpersona_name': kontaktpersona_name,
                    'kontaktpersona_mail': kontaktpersona_mail,
                    'file_category': file_category,
                    'filename': document})

    df_scanned = pd.DataFrame(df_list, columns = ['filename', 'title', 'rekviziti_email', 'kontaktpersona_name', 'kontaktpersona_mail', 'file_category'])

    print(document + " "*100, end="\r") # erase output and print on the same line
```


```python
df_scanned
```

**Step 4. Combine DataFrames together to create the final DataFrame.**


```python
df = pd.concat([df_good, df_scanned])
df
```

**Step 5. Save DataFrame to an Excel file.**


```python
df.to_excel('raw_data.xlsx', index=False)
df.to_csv('raw_data.csv', index=False)
```


```python
print("Files to be inspected manually: ")
print(bad_scanned)
```

    Files to be inspected manually: 
    ['AMASK.docx', 'D4SPII.pdf']
    
