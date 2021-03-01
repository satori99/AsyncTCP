# AsyncTCP 
[![Build Status](https://travis-ci.org/me-no-dev/AsyncTCP.svg?branch=master)](https://travis-ci.org/me-no-dev/AsyncTCP) ![](https://github.com/me-no-dev/AsyncTCP/workflows/Async%20TCP%20CI/badge.svg) [![Codacy Badge](https://api.codacy.com/project/badge/Grade/2f7e4d1df8b446d192cbfec6dc174d2d)](https://www.codacy.com/manual/me-no-dev/AsyncTCP?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=me-no-dev/AsyncTCP&amp;utm_campaign=Badge_Grade)

### Async TCP Library for ESP32 ~~Arduino~~ Espressif IoT Develepment Framework (ESP-IDF) v4.3

This is a fully asynchronous TCP library, aimed at enabling trouble-free, multi-connection network environment for Espressif's ESP32 MCUs.

~~This library is the base for [ESPAsyncWebServer](https://github.com/me-no-dev/ESPAsyncWebServer)~~

## AsyncClient and AsyncServer
The base classes on which everything else is built. They expose all possible scenarios, but are really raw and require more skills to use.

```c
/*
 * AsyncClient Example -- Simple HTTP request
 */
#include <AsyncTCP.h>
#include <esp_log.h>

AsyncClient* client = new AsyncClient();

client->onConnect([](void* arg, AsyncClient* client) {
	ESP_LOGI(TAG, "Connected to %s:%d", inet_ntoa(*client->getRemoteAddress()), client->getRemotePort());
	size_t num_bytes = client->write("GET / HTTP/1.1\r\n"
									 "Host: google.com\r\n"
									 "Connection: close\r\n"
									 "\r\n");
	ESP_LOGI(TAG, "Sending request (%d bytes) ...", num_bytes);
});

client->onAck([](void* arg, AsyncClient* client, size_t len, uint32_t time){
	ESP_LOGI(TAG, "Request was acknowledged (%d bytes in %d ms)", len, (int)time);
});

client->onData([](void* arg, AsyncClient* client, void *data, size_t len) {
	ESP_LOGI(TAG, "Received %d bytes", len);
	printf("%.*s\n", len, (char*)data);
});

client->onTimeout([](void* arg, AsyncClient* client, uint32_t time) {
	ESP_LOGW(TAG, "Client timeout: %d", (int)time);
	client->close();
});

client->onError([](void* arg, AsyncClient* client, int8_t error) {
	ESP_LOGW(TAG, "Client error: %s", client->errorToString(error));
});

client->onDisconnect([](void* arg, AsyncClient* client) {
	ESP_LOGI(TAG, "Client disconnected");
	delete client;
});

ESP_LOGI(TAG, "Connecting to google.com ...");

if (!client->connect("google.com", 80)) {
	ESP_LOGE(TAG, "Failed to connect!");
	delete client;
}
```
