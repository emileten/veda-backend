FROM public.ecr.aws/sam/build-python3.9:latest

WORKDIR /tmp

COPY raster_api/runtime /tmp/raster
RUN pip install "mangum>=0.14,<0.15" /tmp/raster["psycopg-binary"] -t /asset --no-binary pydantic 
RUN rm -rf /tmp/raster

# # Reduce package size and remove useless files
RUN cd /asset && find . -type f -name '*.pyc' | while read f; do n=$(echo $f | sed 's/__pycache__\///' | sed 's/.cpython-[2-3][0-9]//'); cp $f $n; done;
RUN cd /asset && find . -type d -a -name '__pycache__' -print0 | xargs -0 rm -rf
RUN cd /asset && find . -type f -a -name '*.py' -print0 | xargs -0 rm -f
RUN find /asset -type d -a -name 'tests' -print0 | xargs -0 rm -rf
RUN rm -rdf /asset/numpy/doc/ /asset/boto3* /asset/botocore* /asset/bin /asset/geos_license /asset/Misc

COPY raster_api/runtime/handler.py /asset/handler.py

CMD ["echo", "hello world"]