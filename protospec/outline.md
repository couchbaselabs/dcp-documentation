--- stuff in outline that hasn't made it over to the-spec

*   Interfaces

    How there are two sides to this, one how it works with lower
    level protocols and two how it works with user and application
    processes.

*   Operation (Stuff discuss here at a high level on how this
    protocol operates)

    *   Basic Data Transfer (how data gets sent across)
    *   Reliability (ack/nak, recovery etc.)
    *   Flow Control (how do sender and receiver govern how much and
        what is sent)
    *   Multiplexing (not sure if this needed)
    *   Connections (how are they initiated and maintained/managed)
    *   Precedence(state transitions)
    *   Security (how we handle on wire security and any other
        security)

*   Functional Specification

    Typically there is a request, a response and a Payload.

    *   UPR Message Format

        *   Message Type
        *   Message Headers
        *   Message Body
        *   Message Length
        *   Header Fields

    *   Request

        *   Request types
        *   Request Message
        *   Request Methods

            The operation that must be performed

    *   Response

        *   Response types
        *   Response Message/Code

            Typically a status code indicating the result

    *   Payload

        *   Payload Header
        *   Payload Body
        *   Payload Type
        *   Payload Length

*   Method definitions

    *   Types of Methods
    *   Method names (get, set, delete etc.) and their operation

*   Status Code definitions

    *   Success
    *   Warnings/Informational
    *   Client Error
    *   Server Error
    *   Other types

*   Header Field definitions
*   Authentication

    If we intend to have some sort of challenge-response type for
    clients to be authenticated before server responds.

*   Caching

    If we allow caching what is the general approach to correctness of
    cache, validation, age/expiration model etc.)

*   Security (encryption, DOS, any other on the wire issues)
*   Content negotiation (if responses could be ambiguous allow for
    algorithm driven resolution or client driven resolution)

    *   Server driven
    *   Client driven


