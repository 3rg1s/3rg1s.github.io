---
title:  'Ιδωτικότητα και κάμερες IP'
subtitle: 'Ο fuxsocy μας λέει πώς βρήκε ανοιχτές κάμερες στο διαδίκτυο χωρίς κάποια ασφάλεια.'
date: '2020-10-14 12:44:00'
categories: ['privacy']
---

# Πως χάκερς μπορούν να σε δούν μέσω της κάμερας που έχεις στο σπίτι σου.

Στην σημερινή ημέρα όλο περισσότερες κάμερες τοποθετούνται στο σπίτι μας, για την ασφάλεια μας, η για την ασφάλεια κάποιου αγαπημένου μας. Με την σημερινή τεχνολογία οι κάμερες που αγοράζονται περισσότερο είναι αυτές που μπορούμε να έχουμε και πιο γρήγορη πρόσβαση όπου και έαν βρισκόμαστε. Ναι σωστά το σκεφήκατε οι κάμερες αυτές είναι οι λεγόμενες *IP* κάμερες, απο τις οποίες έχουμε πρόσβαση όπου υπάρχει διαθέσιμο δίκτυο δηλαδή παντού. 

Είναι όμως αυτές οι κάμερες ασφαλής απο τον έξω κόσμο? Ας το δούμε....


Μετά από μια γρήγορη αναζήτηση με το [shodan](https://images.shodan.io/?query=country%3A%22GR%22+hipcam), παραπάνω απο 100 κάμερες είναι διαθέσιμες προς παρακολούθηση.

![Screenshot_206](https://user-images.githubusercontent.com/16364370/93470530-94713a80-f8e1-11ea-807e-ff8cbae311ac.png)

Χρησιμοποιώντας το vlc media player και τοποθετόντας στο url bar [](rtsp://IPHERE:554/11) το vlc θα ανοίξει την σύνδεση και θα μας δίνει live μετάδοση βίντεο και ήχου.


![93470957-38f37c80-f8e2-11ea-990a-a6ae6096e183](https://user-images.githubusercontent.com/16364370/96046265-bc14dd80-0e62-11eb-8993-530b8d647585.png)

Οι κάμερες αυτές ονομάζονται hipcam σύμφωνα με τα δεδομένα του shodan.io.

Στην πρώτη φωτογραφία σας δείχνω δεδομένα extracted απο το shodan images, μπορούμε όμως να χρησιμοποιήσουμε και το shodan cli και να κάνουμε και εκεί ένα query και να δούμε πόσες συνολικές κάμερες είναι ανοιχτές(κάποιες μπορεί να χρειάζονται authentication(username,password)).

```
[root@kali-81fv ~]$ shodan count "country:'GR' hipcam"
1524
[root@kali-81fv ~]$ 
```

Αυτές είναι μόνο για της hipcam, μπορούμε επίσης να βάλουμε την πόρτα 554(Real Time Streaming Protocol (RTSP)) η οποία είναι default πόρτα για live μετάδοση βίντεο/εικόνας.

```
[fuxsocy@fuxsocy-81fv ~]$ shodan count "country:'GR' port:554"
16095
[fuxsocy@fuxsocy-81fv ~]$
```

Δεν μπορώ να σας πώ πως μπορείτε να προστατευτείτε διότι δεν γνωρίζω εάν γίνεται να προσθέσουμε στης κάμερες αυτές κάποιο authentication, αυτό που μπορούμε να κάνουμε είναι να της αφαιρέσουμε μέσα απο το σπίτι μας.!