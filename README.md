# Detection Engineering & EDR (Wazuh + YARA Integration)
In questo  progetto ho implementato una soluzione di Endpoint Detection and Response (EDR) integrando il motore di scansione open-source YARA con il SIEM Wazuh su un endpoint Debian 13. L'obiettivo è il monitoraggio predittivo e reattivo di directory sensibili tramite l'analisi automatica dei file basata su firme (pattern matching).
## Architettura e Flusso Logico dei Dati
Il sistema di rilevamento segue una catena di montaggio automatizzata suddivisa in 5 macro-fasi:

File Integrity Monitoring (FIM):  Il modulo syscheck di Wazuh monitora in tempo reale la directory /opt/yara_test/.

Active Response: Al momento della creazione o modifica di un file, Wazuh attiva automaticamente uno script Bash custom (yara.sh), passandogli il percorso del file.

Scansione YARA: Lo script interroga il binario di YARA, che analizza il contenuto del file confrontandolo con le regole di firma caricate (es. test_malware.yar).

Log Forwarding (ossec.conf): Se YARA rileva una minaccia, scrive l'esito nel file /var/ossec/logs/active-responses.log. Wazuh Manager è configurato tramite il tag localfile per monitorare questo log in tempo reale.

Correlazione e Alerting (local_rules.xml): Il modulo di analisi di Wazuh processa il log e fa scattare una regola personalizzata di Livello 13 (Critico) appena intercetta il pattern YARA DETECTED:.

## Configurazione della Regola Custom (Wazuh Manager)
Per elevare la severità dell'evento e generare un allarme rosso visibile in Dashboard, ho inserito una regola personalizzata nel file /var/ossec/etc/rules/local_rules.xml. La regola è identificata dall'ID 100200 con un livello di gravità pari a 13. Il criterio di attivazione si basa sul match testuale della stringa YARA DETECTED:. Quando questa stringa viene intercettata, Wazuh genera un avviso con la descrizione "Allarme EDR: YARA ha rilevato un file malevolo!" e lo classifica sotto i gruppi di sicurezza dedicati a virus, malware_detection e active_response.

## Test Pratico & Proof of Concept (PoC)
Per validare l'intera pipeline di detection, ho simulato l'iniezione di un payload malevolo creando direttamente un file contenente la stringa target configurata nella regola YARA. Per farlo ho eseguito il normale comando: echo "ATTACCO_HACKER_TEST_YARA" > /opt/yara_test/allarme_rosso.txt

## Risultato nei Log di Active Response
Il sistema ha risposto istantaneamente, intercettando la minaccia e scrivendo il match nel diario dei log di sistema. Di conseguenza, nella Dashboard di monitoraggio l'evento è stato correttamente categorizzato con Severity Level 13 (Critical), attivando i canali visivi di triage per gli analisti SOC.

<img width="1828" height="691" alt="Screenshot 2026-06-06 193717" src="https://github.com/user-attachments/assets/7df06cbd-03b8-472a-b40e-8a0030c684d9" />

<br>
<br>
<img width="1847" height="933" alt="Screenshot 2026-06-06 193727" src="https://github.com/user-attachments/assets/a03926dd-5ea3-40bd-8189-1e5c40aa9ebe" />
