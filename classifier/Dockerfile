# image: taccsciapps/classify_image

FROM tensorflow/tensorflow:1.5.0-py3

# pre-reqs
RUN apt-get update && apt-get install -y wget
RUN pip install --upgrade h5py
RUN pip install --no-cache-dir agavepy

# our app
ADD classify_image.py /classify_image.py
ADD entry.sh /entry.sh
RUN chmod +x /entry.sh

ENTRYPOINT ["python", "/classify_image.py", "--bin_message"]
# ENTRYPOINT ["/entry.sh"]
