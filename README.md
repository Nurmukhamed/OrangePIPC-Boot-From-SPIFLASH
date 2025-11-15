# Small howto

I made first commit in February 2022 and then I forgot about this repository.

In 2025, there were some requests about how to get u-boot image, and I had no idea are Dockerfiles working?
Can we build them in 2025?

So I decided to create a Docker images containing u-boot image, push images to Docker Hub and every one can pull them and get u-boot image.

I wish I did it in 2022. 

Also, I refactored Dockerfiles. Because Dockerfiles in 2022 are shame.

# For Orange Pi PC boards

## Get u-boot image

```
sudo docker pull nartykaly/orange-pi-pc-uboot:2022.02

mkdir -p ./output

sudo docker run -it --rm -v ./output:/output nartykaly/orange-pi-pc-uboot:2022.02

ls -al ./output
```

# For Orange Pi Zero Plus boards

## Get u-boot image
```
sudo docker pull nartykaly/orange-pi-zero-plus-uboot:2022.02

mkdir -p ./output

sudo docker run -it --rm -v ./output:/output  nartykaly/orange-pi-zero-plus-uboot:2022.02

ls -al ./output
```


