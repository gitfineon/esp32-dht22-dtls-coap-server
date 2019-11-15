set(srcs
    "coap_server_example_main.c"
    "DHT22.c")

set(includes ".")

idf_component_register(SRCS "${srcs}"
                    INCLUDE_DIRS ${includes}
                    EMBED_TXTFILES certs/coap_ca.pem certs/coap_server.crt certs/coap_server.key)
