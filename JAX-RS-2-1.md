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

## Content Type
@Produces and @Consumes annotation

## Path Params and Query Params
for path param - @PathParam
```
@GET
@Path("employee/{id}")
public Employee getEmployeeById(@PathParam("id") Long id) {
  // implmentation
}

// restricting path template with regex pattern
@GET
@Path("employee/{id: ^[0-9]+$}") // the id path param can only take numbers between 0 to 9
public Employee getEmployeeById(@PathParam("id") Long id) {
  // implmentation
}

// setting default value for path param @DefaultValue
@GET
@Path("employee/{id: ^[0-9]+$}")
public Employee getEmployeeById(@PathParam("id") @DefaultValue("0") Long id) {
  // implmentation
}
```

for query param - @QueryParam
```
@GET
@Path("employee")
public Employee getEmployeeById(@QueryParam("id") Long id) {
  // implmentation
}

// with default value
@GET
@Path("employee")
public Employee getEmployeeById(@QueryParam("id") @DefaultValue("0") Long id) {
  // implmentation
}
```

## Response Object
javax.ws.rs.core.Response

Defines the contract between a returned instance and the runtime when the application needs to provide meta-data to the runtime.

```
@GET
@Path("employee/{id}")
public Response getEmployeeById(@PathParam("id") @DefaultValue("0") Long id) {
  return Response.ok(service.findEmployeeById(id)).status(Response.status.OK).build();
}

// send back the URI of new resource created
@POST
@Path("employees")
public Response createEmployee(Employee employee) 
  service.saveEmployee(employee);
  URI uri = uriInfo.getAbsolutePathBuilder().path(employee.getId().toString()).build();
  return Response.create(uri).status(Response.status.CREATED).build();
}
```

## @Context annotation
The JAX-RS API from Java EE ecosystem provides @Context annotation to inject 12 object instances related to context of HTTP requests.
It behaves just like @Inject

The object instances that it injects are - 
1. Security Context - for current HTTP request
2. Request - used for setting precondition request processing.
3. Application, Configuration & Providers - Provide access to the JAX-RS application, configuration and providers instances.
4. ResourceContext
5. ServletConfig
6. ServletContext
7. HttpServletRequest
8. HttpServletResponse
9. HttpHeaders - maintains the HTTP headers keys and values
10. UriInfo - query parametss and path variables from the URI called.

