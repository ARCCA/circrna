Run Circrna test on Hawk
------------------------
Create a directory to work on::

  mkdir ~/circrna-test
  cd ~/circrna-test

Load required files::

  module purge
  module use /apps/local/modules/projects
  module load nextflow/21.10.6
  module load scwXXXX/kubernetes
  module list

Clone ARCCA Circrna fork::

  git clone https://github.com/ARCCA/circrna.git

The ARCCA team is testing things on the sparrow branch. Confirm
you are in this branch if you want to see our current changes::

  cd circrna
  git branch

At this point you should be able to run Circrna pipeline test::

  nextflow kuberun -r sparrow -latest ARCCA/circrna -profile test,k8s

Access the data::

  nextflow kuberun login -v circrna-data:/data
  cd test_outdir
  ll
  exit

Copy data back to Hawk::

  kubectl cp ubuntu-new-and-improved:/data/hawk.username/test_outdir /scratch/hawk.username/output

Making changes (untested).
---------------
Assuming you have a GitHub account, create a fork. Go to https://github.com/ARCCA/circrna  and click on 'fork', follow the 
instructions to create the fork in your own repo.

- clone your fork repo on Hawk::

  git clone your-repo
  cd your-repo

- make changes, e.g.::

  vim main.nf
  git add main.nf
  git commit -m 'explain the changes made'
  git push

- run kubernetes cluster pointing to your own repo::

  nextflow kuberun -r your-branch -latest your-repo -profile test,k8s
