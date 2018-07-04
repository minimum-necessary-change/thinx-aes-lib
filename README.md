# THiNX AESLib (ESP32, ESP8266, Arduino)

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/8dded023f3d14a69b3c38c9f5fd66a40)](https://www.codacy.com/app/suculent/thinx-aeslib?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=suculent/thinx-aeslib&amp;utm_campaign=Badge_Grade)

An Arduino/ESP32/ESP8266 library to wrap AES encryption with Base64 support. This project is originally based on [AESLib by kakopappa](https://github.com/kakopappa/arduino-esp8266-aes-lib). Unlike the original project actually works and provides optimized methods that do not require using Arduino's flawed String objects.

# Example

```

#include "AESLib.h"

AESLib aesLib;

String plaintext = "12345678;";
int loopcount = 0;

char cleartext[256];
char ciphertext[512];

// AES Encryption Key
byte aes_key[] = { 0x15, 0x2B, 0x7E, 0x16, 0x28, 0xAE, 0xD2, 0xA6, 0xAB, 0xF7, 0x15, 0x88, 0x09, 0xCF, 0x4F, 0x3C };

// General initialization vector (you must use your own IV's in production for full security!!!)
byte aes_iv[N_BLOCK] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 };

// Generate IV (once)
void aes_init() {
  aesLib.gen_iv(aes_iv);
  // workaround for incorrect B64 functionality on first run...
  encrypt("HELLO WORLD!", aes_iv);
}

String encrypt(char * msg, byte iv[]) {  
  int msgLen = strlen(msg);
  char encrypted[2 * msgLen];
  aesLib.encrypt(msg, encrypted, aes_key, iv);  
  return String(encrypted);
}

String decrypt(char * msg, byte iv[]) {
  unsigned long ms = micros();
  int msgLen = strlen(msg);
  char decrypted[msgLen]; // half may be enough
  aesLib.decrypt(msg, decrypted, aes_key, iv);  
  return String(decrypted);
}

void setup() {
  Serial.begin(115200);
  aes_init();
}

void loop() {

  loopcount++;

  sprintf(cleartext, "START; %i \n", loopcount);  

  // Encrypt
  byte enc_iv[N_BLOCK] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 }; // iv_block gets written to, provide own fresh copy...
  String encrypted = encrypt(cleartext, enc_iv);
  sprintf(ciphertext, "%s", encrypted.c_str());
  Serial.print("Ciphertext: ");
  Serial.println(encrypted);

  // Decrypt
  byte dec_iv[N_BLOCK] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 }; // iv_block gets written to, provide own fresh copy...
  String decrypted = decrypt(ciphertext, dec_iv);  
  Serial.print("Cleartext: ");
  Serial.println(decrypted);  

  delay(500);
}

```