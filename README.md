# KMS-Workshop

<b>Pre-requisite</b>

- You must have AWS Account ID
- Cloud9 must be ready
- You must have appropriate user and role to perform this KMS workshop.  Be able to create CMK, create datakey, encrypt and decrypt data using datakey

Step
====
1. Create CMK from AWS Console.  Please name the CMK as 'demo' and create CMK for symmatic key.  You have to select administrator and key's user from user that you create from pre-requisite

2. Create data key by using CMK
<br><b>aws kms generate-data-key --key-id alias/demo --key-spec AES_256 --region ap-southeast-1</b>

<br>Sample output<br>
{<br>
    "CiphertextBlob": "AQIDAHhtAy7pXXMIxPNxuNayt6xCjdjKw84hndoaLSlL3gCSGwFw7Y57twBthh+UoDkU+9H7AAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMsfzWys5jJKRVzO02AgEQgDsN4H73NxjS2K+w+Un88bSDMM+qQcpZI41jspYbnt6pvaEg++daNQoEKQ0j+qRRHgc5j3wmXe0cWdG
XXXXX",<br>
    "Plaintext": "si+XiMuQqSK/BCsQIn0zSaxalEzh1eN6SGv7FB6XXXXX",<br>
    "KeyId": "arn:aws:kms:ap-southeast-1:478263352179:key/8e208876-f685-495f-a6c5-619f1bXXXXX"<br>
}

Plaintext is datakey<br>
CiphertextBlob is datakey that is encrypted by your demo CMK

3. Create data key again.  Now you have to export output into keys.txt  CMK can have multiple datakey
<br><b>aws kms generate-data-key --key-id alias/demo --key-spec AES_256 --region ap-southeast-1 > keys.txt</b>

<br>Sample result in keys.txt
<br>{<br>
    "CiphertextBlob": "AQIDAHhtAy7pXXMIxPNxuNayt6xCjdjKw84hndoaLSlL3gCSGwEOb9dSoOLb92+wdceRRKTzAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMzRtoaCyWyHsjw1YmAgEQgDtvUQgGINuVHE057R9TNZ/XFNKiiDU2SGGgTjKGVYzlZrZ+zFfMdU6zGz7HYxW8f9YOUMVY5LxD70TXXXXX",<br>
    "Plaintext": "RsJTy8azIUUugCwgMF5uQdpc5Ok0oIBDT8FkmvQXXXXX",<br>
    "KeyId": "arn:aws:kms:ap-southeast-1:478263352179:key/8e208876-f685-495f-a6c5-619f1bXXXXX"<br>
}

4. Decode plaintext datakey. The plaintext datakey is in encoded base64.  In order to use datakey, you have to decode it.
<br><b>echo ‘RsJTy8azIUUugCwgMF5uQdpc5Ok0oIBDT8FkmvQXXXXX‘ | base64 --decode > datakey</b>

5. Decode encrypted datakey, the ciphertextBlob.
<br><b>echo 'AQIDAHhtAy7pXXMIxPNxuNayt6xCjdjKw84hndoaLSlL3gCSGwFw7Y57twBthh+UoDkU+9H7AAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMsfzWys5jJKRVzO02AgEQgDsN4H73NxjS2K+w+Un88bSDMM+qQcpZI41jspYbnt6pvaEg++daNQoEKQ0j+qRRHgc5j3wmXe0cWdGXXXXX' | base64 --decode > encrypted-datakey</b>

<b>Encryption</B><br>

6. Now Let's start encrypt the passwords.txt with the datakey that we got from #3.  Encrypt data with datakey
<br><b>openssl enc -in ./passwords.txt -out ./passwords-encrypted.txt -e -aes256 -k fileb://./datakey</b>

7. In real situation, AWS does not recommend to store datakey in Plaintext file. Therefore, remove datakey and passwords.txt

<b>Decryption</B><br>

8. In order to decrypted the passwords-encrypted.txt, we must have datakey.  Now datakey is removed, we have the encrypted datakey version, encrypted-datakey, which encrypted by demo's CMK.   In this task, we will use CMK to decrypted the 'encrypted-datakey' first.
<br><b>aws kms decrypt --ciphertext-blob fileb://./encrypted-datakey</b>

Result<br>
{<br>
    "KeyId": "arn:aws:kms:ap-southeast-1:478263352179:key/8e208876-f685-495f-a6c5-619f1bfXXXXX",<br>
    "Plaintext": "si+XiMuQqSK/BCsQIn0zSaxalEzh1eN6SGv7FB6XXXXX",<br>
    "EncryptionAlgorithm": "SYMMETRIC_DEFAULT"<br>
}

9. Export Plaintext in decode base64 format into datakey file
<br><b>echo ‘i+XiMuQqSK/BCsQIn0zSaxalEzh1eN6SGv7FB6Ykxs=‘ | base64 --decode > datakey</b>

10. Now we already haved datakey file which ready to decrypted the passwords-encrypted.txt
<br><b>openssl enc -in ./passwords-encrypted.txt -out ./passwords-decryptd.txt -d -aes256 -k fileb://./datakey</b>
