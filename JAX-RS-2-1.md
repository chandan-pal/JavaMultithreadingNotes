# JAVA API for Rest Web Services (JAX-RS 2.1)


## REST Archirtecture Constraints
1. Client and Server should be sepearte.
2. Stateless - client and the server does not need to keep state.
3. Cacheable
4. Uniform Interface - The server must expose uniform interface.
5. Layered System - The server can be implemented in layers behind a load balancer


## Application class (javax.ws.rs.core.Application)
Defines the component of a JAX-RS application and supplies additional meta-data. A JAX-RS application or implementation supplies a concrete subclass of this abstract class.

## @ApplicationPath
Identifies the application path that serves as the base URI for all resource URIs provided by **@Path**. May only be applied to **javax.ws.rs.core.Application** or it's extension.

When published in a Servlet container, the value of the application path may be overriden using a servlet-mapping element in the web.xml.

## @Path annotation
Identifies the URI path that a resource class or class method will serve requests for.

## HTTP methods
@GET

@POST

@PUT

@DELETE
