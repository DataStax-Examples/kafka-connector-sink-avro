# datastax-examples-template
This a sample template repo for contributions to the DataStax Examples platform.  This provides the minimum set of items needed to create a new example for submission to the DataStax Examples platform.

## Prerequisites
* Git must be installed
	* If it is not installed then follow the guide found [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).
	


## Cloning this Repo to get started
In a Terminal window.

1) Create a bare clone of the repository.

	git clone --bare https://github.com/bechbd/datastax-examples-template.git`

2) Mirror-push to the new repository.

	cd datastax-examples-template.git
	git push --mirror https://github.com/bechbd/<new repo name>.git`

3) Remove the temporary local repository you created in step 1.

	cd ..
	rm -rf datastax-examples-template.git



4) Clone the newly created repository from step 3.

	git clone https://github.com/bechbd/<new repo name>.git
	
You are now ready to work away in this duplicated repo
 
