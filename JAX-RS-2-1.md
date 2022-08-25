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

## Exception Mappers
ExceptionMapper is a contract for a provider that maps Java exceptions to Response objects so that the client and see a proper response and make some sense out of it.

```
@Provider
public class MyExceptionMapper implements ExceptioMapper<MyException> {
  @Override
  public Response toResponse(MyException ex) {
    return Response.status(Status.BAD_REQUEST).entity(ex.getMessage()).build();
  }
}
```

## @HeaderParam
to grab property from header and inject into request call
```
@GET
@Path("employee/{id}")
public Response getEmployeeById(@PathParam("id") @DefaultValue("0") Long id, @HeaderParam("Referer")) {
  return Response.ok(service.findEmployeeById(id)).status(Response.status.OK).build();
}
```

## Caching with Cache control and eTag
JAX-RS provides contract for managing caching. The `cache-control` header can be set on the response object to define HTTP caching.
Java EE provide **CacheControl** class to achieve the same.


```
@Path("/book/{id}")
@GET
public Response getBook(@PathParam("id") long id){

    Book myBook = getBookFromDB(id);

    CacheControl cc = new CacheControl();
    cc.setMaxAge(86400);
    cc.setPrivate(true);

    ResponseBuilder builder = Response.ok(myBook);
    builder.cacheControl(cc); // this will set cache-control header in response
    return builder.build();
}
```

EntityTag (ETag) - along with `cache-control` header and `ETag` header can be passed in the response to validate the resource which the client has.\
The ETag should be a unique hash of the resource and requires knowledge of internal business logic to construct. One possiblity is to use the hashcode of the object as ETag.

```
@Path("/book/{id}")
@GET
public Response getBook(@PathParam("id") long id, @Context Request request){

    Book myBook = getBookFromDB(id);
    CacheControl cc = new CacheControl();
    cc.setMaxAge(86400);

    EntityTag etag = new EntityTag(Integer.toString(myBook.hashCode()));
    ResponseBuilder builder = request.evaluatePreconditions(etag);

    // cached resource did change -> serve updated content
    if(builder == null){
        builder = Response.ok(myBook);
        builder.tag(etag);
    }

    builder.cacheControl(cc);
    return builder.build();
}
```
A `ResponseBuilder` is automatically constructed by calling **evaluatePreconditions** on the request.\
If the builder is null, the resource is out of date and the object needs to be sent back in the response.\
Otherwise, the preconditions indicate that the client has the latest version of the resource and the 304 Not Modified status code will be automatically assigned.


## JAX-RS File Upload
```
@POST
@Path("/upload")
@Consumes({MediaType.APPLICATION_OCTET_STREAM, "image/png", "image/jpeg", "image/gif"})
public Response uploadPicture(File picture, @QueryParam("id") @NotNull Long id) {
  Employee employee = service.findEmployeeById(id);
  try (Reader reader = new FileReader(picture)) {
    employee.setPicture(Files.readAllBytes(Paths.get(picture.toURI())));
    service.saveEmployee(employee);
    
    Response.ok().build();
  } catch (IOException e) {
    e.printStackTrace();
    return Response.serverError().build();
  }
}
```

## JAX-RS File Download
```
@GET
@Path("download")
@Produces({MediaType.APPLICATION_OCTET_STREAM, "image/png", "image/jpeg", "image/gif"})
public Response getEmployeePicture(@QueryParam("id") @NotNull Long id) throws Exception {
  Employee employee = service.findEmployeeById(id);
  if (employee!=null) {
    return Response.ok().entity(Files.write(Paths.get("ppic.png"), employee.getPicture()).toFile()).build();
  }
  return Response.noContent().build();
}
```

## Request & Response Filters
These are like interceptors which can be placed to perform some operations before a requets is processed by a resource controller or before a resonse is sent to the client.

**ContainerResponseFilter**
```
// static container response filter
@Provider
public class CacheResponseFilter implement ContainerResponseFilter {
  @Override
  public void fiter(ContainerRequestContext requestContext, ContainerResponseContext responseContext) throws IOException {
    String method = requestContext.getMethod();
    String path = requestContext.getUriInfo().getPath();
    if ("GET".equalsIgnoreCase(method) && "employees".equalIgnoreCase(path)) {
      CacheControl cacheControl = new CacheControl();
      cacheControl.setMaxAge(100);
      cacheControl.setPrivate(true);
      
      responseContext.getHeaders().add("Cache-Control", cacheControl);
      responseContext.getHeaders().add("MyMessage", "Cache Control added by filter!");
    }
  }
}
```

**Dynamic Response Filter - Annotation based**
```

// custom annotation MaxAge creation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MaxAge {
  int age();
}

public DynamicFilter implements ContainerResponseFilter {
  int age;
  
  public DynamicFilter(int age) {
    this.age = age;
  }
  
  public DynamicFilter() {
  
  }
  
  @Override
  public void fiter(ContainerRequestContext requestContext, ContainerResponseContext responseContext) throws IOException {
    String method = requestContext.getMethod();
    String path = requestContext.getUriInfo().getPath();
    if ("GET".equalsIgnoreCase(method)) {
      CacheControl cacheControl = new CacheControl();
      cacheControl.setMaxAge(age);
      
      responseContext.getHeaders().add("Cache-Control", cacheControl);
    }
  }
  
  
  // implement dynamic filter register
  @Provider
  public class DynamicFilterFeature implements DynamicFeature {
    @Override
    public void configure(ResourceInfo resourceInfo, FeatureContext context) {
      MaxAge annotation = resourceInfo.getResourceMethod().getAnnotation(MaxAge.class);
      
      if (annotation != null) {
        DynamicFilter dynamicFilter = new DynamicFilter(annotation.age());
        context.register(dynamicFilter);
      }
    }
  }
  
  // use the annotation based dynamic filter
  @GET
  @Path("id")
  @Produces("application/json")
  @MaxAge(age = 200)
  public Respinse getEmployeeById(@PathParam("id") Long id) {
    //
    return Response.ok().status(Status.OK).build();
  }
  
}
```

**ContainerRequestFilter**
```
@Provider
@PreMatching // filter is executed before the request is matched to the controller
public class PreMatchingServerRequestFilter implements ContainerRequestFilter {
  
  @Override
  public void filter(ContainerRequestContext requestContext) throws IOException {
    // some implementation
  }
}
```

## JAX-RS Async
@Suspended annotation and asyncResponse

```
@Inject
ManagedExecutorService managedExecutorService; // extension of executor service


@POST
@Path("run")
public void run(@Suspended AsyscResponse asyncResponse) {
  // get hold of the current thread (as the current thread will be suspended and we will spawn new thread for heavy task
  final String currentThread = Thread.currentThread().getName();
  
  // set time out for asyncResponse
  asyncResponse.setTimeout(5000, TimeUnit.MILISECONDS); // set time out duration
  
  // optional custom timeout handler
  asyncRespinse.setTimeoutHandler(response -> {
    response.resume(Response.status(Status.REQUEST_TIMEOUT).entity("Sorry, the request timed out. Please try again.").build());
  });
  
  // Register other callbacks
  asyncResponse.register(CompletionCallbackHandler.class);
  
  managedExecutorService.submit(()-> {
    final String spawnedThreadName = Thread.currentThread().getName();
    
    // very expensive task
    service.compute();
    
    // resume the main thread and send response
    asyncResponse.resume(Response.ok().header("Original Thread", currentThread))
      .header("Spawned Thread", spawnedThreadName)
      .status(Status.OK).build());
  });
}
```


## JSON-B Integration

## JSON-P Integration
JSON Processing API

It is a java API to process (e.g. parse, generate, transform and query) JSON messages.\
It produces and consumes JSON text in a streaming fashion and allows to build a java object model for JSON text using API classes.


## JAX - Client API
This API can be used to call downstream API from within a server

```
@RequestScoped
public class JaxRsClient {
  private Client client;
  WebTarget webTarget;
  
  private final String haveIBeenPawned = "https://haveibeenpwned.com/api/v2/breachedaccount";
  
  @PostConstruct
  private void init() {
    client = ClientBuilder.newBuilder().connectTimeout(7, TimeUnit.SECONDS).readTimeout(3, TimeUnit.SECONDS).build();
    
    webTarget = client.target(haveIBeenPawned);
  }
  
  @PreDestroy
  private void destroy() {
    if (client != null) {
      client.close();
    }
  }
  
  
  public int checkBreaches() {
    JsonArray jsonValues = webTarget.path("account).resolveTemplate("account", email).request(MeadiaType.TEXT_PLAIN).get(JsonArray.class);
    parseJsonArray(jsonValues);
    return jsonValues.size();
  }
}
```
