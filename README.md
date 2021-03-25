# KMS-Workshop

<b>Pre-requisite</b>

- You must have AWS Account ID
- Cloud9 must be ready
- You must have appropriate user and role to perform this KMS workshop.  Be able to create CMK, create datakey, encrypt and decrypt data using datakey

Step
====
1. Create CMK from AWS Console.  Please name the CMK as 'demo' and create CMK for symmatic key.  You have select administrator and key's user from user that you create from pre-requisite

2. Create data key by using CMK
aws kms generate-data-key --key-id alias/demo --key-spec AES_256 --region ap-southeast-1

Sample output
{
    "CiphertextBlob": "AQIDAHhtAy7pXXMIxPNxuNayt6xCjdjKw84hndoaLSlL3gCSGwFw7Y57twBthh+UoDkU+9H7AAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMsfzWys5jJKRVzO02AgEQgDsN4H73NxjS2K+w+Un88bSDMM+qQcpZI41jspYbnt6pvaEg++daNQoEKQ0j+qRRHgc5j3wmXe0cWdG
XXXXX",
    "Plaintext": "si+XiMuQqSK/BCsQIn0zSaxalEzh1eN6SGv7FB6XXXXX",
    "KeyId": "arn:aws:kms:ap-southeast-1:478263352179:key/8e208876-f685-495f-a6c5-619f1bXXXXX"
}

Plaintext is datakey
CiphertextBlob is datakey that is encrypted by your demo CMK

3.Create data key again.  Now you have to export output into keys.txt  CMK can have multiple datakey
aws kms generate-data-key --key-id alias/demo --key-spec AES_256 --region ap-southeast-1 > keys.txt

Sample result in keys.txt
{
    "CiphertextBlob": "AQIDAHhtAy7pXXMIxPNxuNayt6xCjdjKw84hndoaLSlL3gCSGwEOb9dSoOLb92+wdceRRKTzAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMzRtoaCyWyHsjw1YmAgEQgDtvUQgGINuVHE057R9TNZ/XFNKiiDU2SGGgTjKGVYzlZrZ+zFfMdU6zGz7HYxW8f9YOUMVY5LxD70TXXXXX",
    "Plaintext": "RsJTy8azIUUugCwgMF5uQdpc5Ok0oIBDT8FkmvQXXXXX",
    "KeyId": "arn:aws:kms:ap-southeast-1:478263352179:key/8e208876-f685-495f-a6c5-619f1bXXXXX"
}

3. Decode plaintext datakey. The plaintext datakey is in encoded base64.  In order to use datakey, you have to decode it.
echo ‘RsJTy8azIUUugCwgMF5uQdpc5Ok0oIBDT8FkmvQXXXXX‘ | base64 --decode > datakey

4. Decode encrypted datakey, the ciphertextBlob.
echo 'AQIDAHhtAy7pXXMIxPNxuNayt6xCjdjKw84hndoaLSlL3gCSGwFw7Y57twBthh+UoDkU+9H7AAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMsfzWys5jJKRVzO02AgEQgDsN4H73NxjS2K+w+Un88bSDMM+qQcpZI41jspYbnt6pvaEg++daNQoEKQ0j+qRRHgc5j3wmXe0cWdGXXXXX' | base64 --decode > encrypted-datakey

5. Now Let's start encrypt the passwords.txt with the datakey that we got from #3.  Encrypt data with datakey
openssl enc -in ./passwords.txt -out ./passwords-encrypted.txt -e -aes256 -k fileb://./datakey

6. In real situation, AWS does not recommend to store datakey in Plaintext file. Therefore, remove datakey and passwords.txt
7. In order to decrypted the passwords-encrypted.txt, we must have datakey.  Now datakey is removed, we have the encrypted datakey version, encrypted-datakey, which encrypted by demo's CMK.   In this task, we will use CMK to decrypted the 'encrypted-datakey' first.
aws kms decrypt --ciphertext-blob fileb://./encrypted-datakey

Result
{
    "KeyId": "arn:aws:kms:ap-southeast-1:478263352179:key/8e208876-f685-495f-a6c5-619f1bfXXXXX",
    "Plaintext": "si+XiMuQqSK/BCsQIn0zSaxalEzh1eN6SGv7FB6XXXXX",
    "EncryptionAlgorithm": "SYMMETRIC_DEFAULT"
}

8. Export Plaintext in decode base64 format into datakey file
echo ‘i+XiMuQqSK/BCsQIn0zSaxalEzh1eN6SGv7FB6Ykxs=‘ | base64 --decode > datakey

9. Now we already haved datakey file which ready to decrypted the passwords-encrypted.txt
openssl enc -in ./passwords-encrypted.txt -out ./passwords-decryptd.txt -d -aes256 -k fileb://./datakey
