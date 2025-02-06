# 2025-02-06-DSCI310
### Even More on Docker:

```bash
docker run \
    --rm \
    -it
    -e PASSWORD="password" \
    -p 8787:8787 \
    - v /$(pwd):/home/rstudio/pizza
    rocker/rstutio:4.4.2
 ```
 
> This command is to be run in the terminal. Same as when we don't have all the \ and indentation.
> This format is really for better viewing.
> - The most important part of this is really `-e PASSWORD="password"` and `-p 8787:8787`.
> - `-v` (dash volume) is take the current directory `pizza` and "mount" it onto the container folder--allows us to get one-to-one synchronization between the local and the container.

- `yml.` file makes this current command and save it in a way that docker understands.
> If you try to copy and paste this command into terminal, you won't be able to edit it in the terminal, but you'd able to actively edit it within the .yml file:).
> Create `docker-compose.yml`.

All this docker compose yml does is taking this command and takes pieces and components from the command:
- First line `services` tells docker what it wants to include with a 'special name'. We'd want to specify this name manually: 'analysis-env'
**The `tidyverse` container image doesn't work with MacOS:(, but we can just work with the RStudio containe**
- Order doesn't really matter with image, ports.
- `yml` is a little bit weird in a sense that sometimes it uses dashes (-) for listing (typically if the parameter can take in multiple values, like for `ports:`), sometimes it doesn't.
- For `volumes:`, we don't need to include `/$(pwd)` component.
    - Volumes are for **what we want to mount**.
    - Take all folders we have in our current directory, and create a new folder and shove everything in it.
    - If `pizza` is already in the home directory, this mounted `pizza` will overwrite it.
    - You typically **DON'T** see mounting into the home directory directly, this is why we typically want to mount into a directory that is not `home`.
- For `environment:`
    - Will differ depending on different containers.
    - Note: because we do want to version control this `yml` file, the fact that we have a password here isn't ideal. This is okay in this containers
        -  BUT: DO NOT put your personal access token into these file!! 

services: 
    analysis-env: 
        image: rocker/rstudio:4.4.2
        ports:
            - "8787:8787"
        volumes:
            - .:/home/rstudio/pizza
        environment:
            PASSWORD: "password"

> This docker compose file will completely reproduces the docker command!! (Yaay)
- We can run **`docker compose up`** in our terminal!
- We can then run our container in our browser and see all the same container.
- Now we know how to store and share docker file through this expansion of the previous docker command:), it's no longer specific to one operating systems between windows/MacOS/linux.

In the Rocker documentation, there's the variable **`DISABLE_AUTH`** and set it to true:
- Bypasses the login where you no longer need to log in the port service.
- How do we translate to yml?
    - This parameter will go under the environment:

    services: 
    analysis-env: 
        image: rocker/rstudio:4.4.2
        ports:
            - "8787:8787"
        volumes:
            - .:/home/rstudio/pizza
        environment:
            PASSWORD: "password"
            **DISABLE_AUTH: true**

**Note not to mess up the identation of the yml. file. Consult class note for an example yml. file for exam purposes!**

- We can also run 2 containers at the same time, but we have to give them different names:

services: 
    analysis-env: 
        image: rocker/rstudio:4.4.2
        ports:
            - "**8787**:8787"
        volumes:
            - .:/home/rstudio/pizza
        environment:
            PASSWORD: "password"

    analysis-env2:
        image: rocker/rstudio:4.4.2
        ports:
            - "**8788**:8787"
        volumes:
            - .:/home/rstudio/pizza
        environment:
            PASSWORD: "password"

- In terminal, run also **`docker compose rm`** to ensure we're out of all the ports.

- In Docker world, everything has to be run through bash command.
- What happens if we want other packages in the container?
- e.g.) Let's say we want to install another R package (e.g., Git), but I don't want to go into the container every other time to re-install the same package over and over.
    -  That's to day, we want to customize our container.

    - We can create a new file under our current lecture folder called `Dockerfile`:

In our `Dockerfile`:
FROM rocker/rsudio:4.4.2

> Next, in the terminal, run **`docker build --tag mycontainer .`** to build the container.
- Everytime we edited the Dockerfile, we would have to rerun to rebuild the container. **Make sure this step of rebuilding is done repeatedly.**
    - `.` specifies that all files are starting from the current working directory. 
    - When we run this, the Docker build system will lock at the Dockerfile and build a container based on this file line-by-line.
    - Now we'll hav eour new container called `mycontainer` after we run this:)!

Now how do we use this container?
- We can edit the compose file and instead of having `image: rocker/rstudio:4.4.2`, we can now instead write: `image: mycontainer`.
- The Dockerfile specifies only the `image`, as such, all the rest of the `.yml` docker-compose file is still needed:

services: 
    analysis-env: 
        image: mycontainer
        ports:
            - "**7777**:8787"
        volumes:
            - .:/home/rstudio/pizza
        environment:
            PASSWORD: "password"

- Running `localhost/7777` in browser will now run the container we've specified in the Dockerfile.

For example now, we want `renv` in this container, but how do we customize our container such that it allows `renv`?
- In internal, we can install new package by running: `Rscript -e <R code>` 
    - For example: `Rscript -e <R code> "print('hello')"`.
- In Dockerfile:
> The `repos=` parameter is needed because the user need manually run these command to pre-select the mirror of the package since there can't be any user input during the installation process to select the mirror (non-interactive mode).

> FROM rocker/rsudio:4.4.2
> 
> RUN Rscript -e "install.packages('renv', repos = c(CRAN = 'https://cloud.r-project.org'))"

#### Homework: 
install specific package in the repository with a specific package version:
RUN Rscript -e "install.packages('renv', repos = c(CRAN = 'https://cloud.r-project.org'))"
RUN Rscript -e "install.packages('remotes', repos = c(CRAN = 'https://cloud.r-project.org'))"

- We can create the renv system locally on the computer, and then we use this renv with Docker (serarch up the Docker documentation in R, read / copy Appraoch one and follow instruction there beginning from `COPY renv.lock /home/rsudio/renv.lock`):
- Create a lock file and copy:

> FROM rocker/rsudio:4.4.2
> 
> RUN Rscript -e "install.packages('renv', repos = c(CRAN = 'https://cloud.r-project.org'))"
> RUN Rscript -e "install.packages('remotes', repos = c(CRAN = 'https://cloud.r-project.org'))"
> 
> COPY renv.lock /home/rsudio/renv.lock


### How do I take this container (and image) from my local and share it?
#### Docker Hub:
- When we use rocker/rstudio:4.4.2, there's a service within rocker with the repo 'rstudio' that can be shared.
- First we need to login to docker with the command `docker login -u chendanialy`
    - `-u` stands for user name.
    - Don't just type in `docker login alone`.

- To get our docker up to the hub, we will change our way of building our container by including our username:
> `docker build --tag **chendaniely**/mycontainer .`
- When a command gets changed, Docker will only run the added command (yayy) so that it doesn't re-run the previously 'loaded' package again.

Then finally run:
> `docker push chendanialy/mycontainer`

- We can then be able to see our container on dockerhub when we search up our own username (e.g., `chendanialy` in this example).
- When we use this container with this image, we need to make sure we build the edited container, push it onto Docker Hub so that the new image is now .


### How to automate the build process?
