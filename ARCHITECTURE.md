# Architecture: psr-http-message-bridge

## Purpose

A Symfony bridge that converts between Symfony's `HttpFoundation` request/response objects and PSR-7 (`psr/http-message`) objects. Allows Symfony applications to accept PSR-7 `ServerRequestInterface` in controllers and to return PSR-7 `ResponseInterface` from them.

## Directory Structure

```
Factory/
  PsrHttpFactory.php           - Converts HttpFoundation Request/Response → PSR-7 objects
  HttpFoundationFactory.php    - Converts PSR-7 Request/Response → HttpFoundation objects
  UploadedFile.php             - PSR-7 UploadedFileInterface wrapper for HttpFoundation files
HttpFoundationFactoryInterface.php  - Contract for PSR-7 → HttpFoundation conversion
HttpMessageFactoryInterface.php     - Contract for HttpFoundation → PSR-7 conversion
ArgumentValueResolver/
  PsrServerRequestResolver.php - Symfony ArgumentValueResolver: injects PSR-7 ServerRequest into controllers
EventListener/
  PsrResponseListener.php      - Symfony event listener: converts PSR-7 Response to HttpFoundation Response
Tests/                         - PHPUnit unit and functional tests
```

## Key Design Decisions

- **Bidirectional conversion**: Two separate factory classes handle each direction of conversion, keeping each class focused on a single transformation direction.
- **ArgumentValueResolver integration**: Controllers can type-hint `ServerRequestInterface` as a parameter and the resolver automatically converts the incoming Symfony `Request`.
- **Event listener for responses**: A `kernel.view` event listener converts PSR-7 responses returned by controllers into `HttpFoundation\Response` objects that Symfony can send.
- **PSR-17 factory injection**: `PsrHttpFactory` requires a PSR-17 factory (e.g., Guzzle's `HttpFactory` or Nyholm's `Psr17Factory`) to create PSR-7 objects, keeping the bridge PSR-7-implementation-agnostic.

## Extension Points

- Pass a different PSR-17 factory implementation to `PsrHttpFactory` to change which PSR-7 library is used.
- Implement `HttpMessageFactoryInterface` for custom conversion logic.

## Dependency Flow

```
Symfony controller (type-hinted ServerRequestInterface)
  └─> PsrServerRequestResolver
        └─> PsrHttpFactory::createRequest(HttpFoundation\Request)
              └─> PSR-17 factory (creates PSR-7 objects)

Controller returns PSR-7 ResponseInterface
  └─> PsrResponseListener (kernel.view)
        └─> HttpFoundationFactory::createResponse(PSR-7 Response)
              └─> HttpFoundation\Response
```
