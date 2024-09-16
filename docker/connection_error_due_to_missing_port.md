# ConnectionError due to missing port

Not so much about docker, but about one of those moments where I stare too hard at the code and just need an extra pair of eyes to point out the problem.

I was trying to make an api request to the main application from Jupyter notebook, but kept getting an error:

```python
ConnectionError: HTTPConnectionPool(host='localhost', port=8000): Max retries exceeded with url: /api/endpoint/ (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0xffff48cd2260>: Failed to establish a new connection: [Errno 111] Connection refused'))
```

I couldn't figure out why the connection failed. The application was up and running, pgAdmin was up and running, and Django shell was able to connect to the db and make queries. I asked my teammates, and they noticed that Jupyter was running in a docker container, and suggested that perhaps the port might not be open. They were right! I was able to make the query in python shell on the host machine to check that the main application would receive the request. I was able to create the port mapping in docker-compose.yml.