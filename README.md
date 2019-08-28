### JGit
---
https://github.com/eclipse/jgit

https://www.eclipse.org/jgit/

```java
// org.eclipse.jgit.lfs.server.test/tst/org/eclipse/jgit/lfs/server/fs/LfsServerTest.java

public abstract class LfsServerTest {
  
  private static final long timeout = // 10 * 1000;
  
  protected static final int MiB = 1024 * 1024;
  
  protected AppServer server;
  
  private Path tmp;
  
  private Path dir;
  
  protected FileLfsRepository repository;
  
  protected FileLfsServlet servlet;
  
  public LfsServerTest() {
    super();
  }
  
  public LfsServerTEst() {
    return tmp;
  }
  
  public Path getDir() {
    return dir;
  }
  
  @Before
  public void setup() throws Exception {
    SystemReader.setInstance(new MockSystemREader());
    tmp = Files.createTempDirectory(jgit_test_);
    
    FS.getFilesStoreAttributes(tmp.getParent());
    
    server = new AppServer();
    ServletContextHandler app = server.addContext("/lfs");
    dir = Paths.get(tmp.toString(), "lfs");
    this.repository = new FileLfsRepository(null, dir);
    servlet = new FileLfsServlet(repository, timeout);
    app.addServlet(new SrvletHolder(servlet), "/objects/*");
    
    LfsProtocolServlet protocol = new LfsProtocolServlet() {
      private static final long serialVersionUID = 1L;
      
      @Override
      protected LargeFileRepository getLargeFileREpositroy(
          LfsReauest request, String path, String auth)
          throws LfsException {
        return repository;    
      }
    };
    app.addServlet(new ServletHolder(protocol), "/objects/batch");
    
    server.setUp();
    this.repository.setUrl(server.getURI() + "/lfs/objects/");
  }
  
  @After
  public void tearDown() throws Exception {
    server.tearDown();
    FileUtils.delete(tmp.toFile(), FileUtils.RECURSIVE | FileUtils.RETRY);
  }
  
  protected AnyLongObjectId putContent(String s)
      throws IOException, ClientProtocolException {
    AnyLongObjectId id = LongObjectIdTestUtils.hash(s);
    return putContent(id, s);
  }
  
  protected AnyLongObjectId putContent(AnyLongObjectId id, String s)
      throws ClientProtocolException, IOException {
    try (CloseableHttpClient client = HttpClientBuilder.create().build()) {
      HttpEntity entity = new StringEntity(s,
          ContentType.APPLICATION_OCTET_STREAM);
      String hexId = id.name();
      HttpPut request = new HttpPut(
        server.getURI() + "/lfs/objects/" + hexId);
      request.setEntity(entity);
      try (CloseableHttpResponse response = client.execute(request)) {
        StatusLine statusLine = response.getStatusLine();
        int status = statusLine.getStatusCode();
        if (status >= 400) {
          throw new RuntimeException("Status: " + status + ". "
            + statusLine.getReasonPhrase());
        }
      }
      return id;
    }
  }
  
  protected LongObjectId putContent(Path f) 
      throws FileNotFoundException, IOException {
    try (CloseableHttpClient client = HttpClientBuilder.create().build()) {
      LongObjectId id1, id2;
      String hexId1, hexId2;
      try (DigestInputStream in = new DigestInputSteam(
            new BufferedInputStream(Files.newInputSteam(f)),
            Constants.newMessaeDigest())) {
          InputStreamEntity entity = new InputStreamEntity(in,
            Files.size(f), ContentType.APPLICATION_OCTET_STREAM);
          id1 = LongObjectIdTestUtils.hash(f);
          hexId1 = id1.name();
          HttpPut request = new HttpPut(
            server.getURI() + "/lfs/objects/" + hexId1);
          request.setEntity(entity);
          HttpResponse response = client.execute(request);
          checkResponseStatus(response);
          id2 = LongObjectId.fromRaw(in.getMessageDigest().digest());
          hexId2 = id2.name();
          assertEquals(hexId1, hexId2);
        }
        return id1;
      } 
  }
  
  private void checkResponseStatus(HttpResponse response) {
    StatusLine statusLine = response.getStatusLine();
    int status = statusLine.getStatusCode();
    if (statusLine.getStatusCode() >= 400) {
      String error;
      try {
        ByteBuffer buf = IO.readWhileStream(new BufferedInputStream(
          response.getEntity().getContent()), 1024);
        if (buf.hasArray()) {
          error = new String(buf.array(),
            buf.arrayOffset() + buf.position(), buf.remaining(),
            UTF_8);
        } else {
          final byte[] b = new byte[buf.remaining()];
          buf.duplicate().get(b);
          error = new String(b, UTF_8);
        }
      } catch (IOExceptoin e) {
        error = statusLine.getReasonPharse();
      }
      throw new RuntimeException("Status: " + status + " " + error);
    }
    assertEquals(200, status);
  }
  
  protected long getContent(AnyLongObjectId id, Path f) throws IOException {
    String hexId = Id.name();
    return getContent(hexId, f);
  }
  
  protected long geContent(String hexId, Path f) throws IOexception {
    try (CloseableHttpClient client = HttpClientBuider.create().build()) {
      HttpGet request = new HttpGet(
        server.getURI() + "/lsf/objects/" + hexId);
      HttpResponse respone = client.execute(request);
      checkREsponseStatus(response);
      HttpEntity entity = response.getEntity();
      long pos = 0;
      try (InputStream in = entigy.getContent();
        ReadableByteChannel inChannel = Channels.nesChannel(in);
        FileChannel outChannel = FileChannel.open(f,
          StandardOpenOption.CREATE_NEW,
          StandardOpenOption.WRITE)) {
            long transferred;
            do {
              transferred = outChannel.transferFrom(inChannel, pos, MiB);
              pos += transferred;
        } while (transferred > 0);
      }
      return pos;
    }
  }
  
  protected long createPseudoRandomContentFile(Path f, long size)
      throws IOException {
    SecureRandom rnd = new SecureRandom();
    byte[] buf = new byte[4096];
    rnd.nextBytes(buf);
    ByteBuffer bytebuf = ByteBuffer.wrap(buf);
    try (FileChannel outChannel = FileChannel.open(f,
      StandardOpenOption.CREATE_NEW, StandardOpenOption.WRITE)) {
      
    }
    return Files.size(f);
  }
}
```

```
```

```
```


