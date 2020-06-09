---
title: "Lesson 1: Core concepts using an analysis workflow example"
teaching: 30
exercises: 60
questions:
- "What are the different considerations for reproducible analysis?"
objectives:
- Learn core elements of reproducible analysis
- Familiarize with various technology terms
keypoints:
- Reproducible analysis is technologically possible
- Learning these technologies can help produce more reliable research output
- Using such frameworks provide a better way to communicate information to colleagues and collaborators
---

> ## You can skip this lesson if you can answer these questions? --->
>
>  - What are the four core elements of a reproducible analysis?
>  - Why should you annotate your data in a machine accessible manner?
>  - Why should you use version control for data and code?
>  - What are some ways to share your analysis environment with others?
>  - What does continuous integration help you with?
{: .challenge}

This lesson uses the [Simple Workflow](https://github.com/repronim/simple_workflow)
Github repository to illustrate core concepts of reproducible analysis
and pitfalls associated with complex data, software, and computing
environments. The complete simple workflow paper is available
[here](https://f1000research.com/articles/6-124/).

### Lesson outline

- [Overview of the workflow](#overview)
- [Element 1: Storing data and metadata](#element1)
   - CSV, JSON/LD, BIDS, NIDM-E
- [Element 2: Creating a reproducible execution environment](#element2)
   - Virtual machines, Containers, Scripts
- [Element 3: Running analysis and storing expected results and provenance](#element3)
   - Determine which execution outputs constitute test
- [Element 4: Checking output consistency using continuous integration](#element4)
   - Couple with continuous integration (CI) services such as [CircleCI](https://circleci.com/) or [travis](https://travis-ci.org/), or
    with your own CI servers running [jenkins](https://jenkins.io/).
- [Results from running the Workflow](#results)

### Lesson requirements

You will make the most of this lesson if you have an understanding of:

- [Unix shell](http://swcarpentry.github.io/shell-novice/)
- [Version control](http://www.reproducibleimaging.org/module-reproducible-basics/02-vcs/)

<a name="overview" />

### Overview of workflow
The basic workflow presented here (i) extracts a collection of brain images
and associated phenotypic trait (e.g., age) from a spreadsheet, and (ii) runs
a Nipype workflow that takes the anatomical brain images and performs some
simple anatomical image processing. Executing the whole workflow may take a
bit of time to run depending on the power of your machine or cluster. For
learning purposes and to minimize the time you can run the workflow on one
participant.

> ## Hands on exercise:
>
>  Can you rerun the analysis in the simple workflow example?
>
> > ## Solution
> >
> > Follow the README in the repo and rerun the analysis with the
> >  [Docker example.](https://github.com/repronim/simple_workflow#1-to-execute-demo-with-docker)
> {: .solution}
{: .challenge}

<a name="element1" />

### Element 1: Storing data and metadata
To ensure reproducibility all data and metadata must be accessible and
preferably be machine-accessible.

> ## Machine accessibility

> Machine accessibility means that information regarding an analysis or
> research workflow (a.k.a., metadata) can be easily accessed by and parsed with automated tools.
> Typically, the main approach to describe our research is to write a document
> that is shared with colleagues and collaborators. However, extracting
> relevant information regarding data acquisition, processing and/or
> analysis, requires significant human resources, both to interpret and >
> translate into code. An alternate approach is to encode the metadata > using
> structured markup (e.g., [RDF](https://www.w3.org/TR/rdf11-concepts/), >
> [JSON](http://www.json.org/), [XML](https://www.w3schools.com/xml/)). Often >
> such markup can be standardized to provide machine accessibility. {: .callout}

In this example data and metadata are stored in a
[google spreadsheet](https://docs.google.com/spreadsheets/d/11an55u9t2TAf0EV2pHN0vOd8Ww2Gie-tHp9xGULh_dA).
Phenotypic information is stored as characters/strings. The imaging data are stored as
pointers/links to files in the NITRC XNAT repository. However, this particular example
does not have any semantic or (data) type information associated with the input file.

The column headers can be described in detail in a JSON document using
[JSONLD](https://json-ld.org/) a format that supports semantic annotation. The
annotation provides information about the data contained in the column and
allows for the harmonization of the information with other similar tables. For example, the
JSONLD metadata key could tell us that the URLs correspond to anatomical
T1-weighted images of the human brain and that the age of participants is
expressed in years.


[Lesson 2](../03-data) covers different aspects of data annotation,
harmonization, cleaning, storage, and sharing.

> ## Hands on exercise:
>
>  What types of output files are created by the workflow?
>
> > ## Solution
> >
> > There are four types of output files created:
> >
> > 1. Compressed [Nifti files](https://brainder.org/2012/09/23/the-nifti-file-format/)
> >    corresponding to different segmentations
> > 2. A JSON file corresponding to the volume measures and voxel counts of
> >    brain segmentations.
> > 3. [VTK surface files]() for each subcortical surface
> > 4. A [Turtle syntax](https://www.w3.org/TR/turtle/) provenance file capturing
> >    the information
> {: .solution}
>
>  What are some of the drawbacks of the form in which the input and output are
>   represented?
>
> > ## Solution
> >
> > The input Google spreadsheet does not provide a key to annotate each column.
> > The output JSON file also does not describe the keys anywhere. These things
> > make it difficult for a human or a machine to interpret the values. In addition,
> > it prevents harmonization of data.
> {: .solution}
{: .challenge}

<a name="element2" />

### Element 2: Creating a reproducible execution environment
The second component of this example is a setup script and a Docker container
that creates the necessary computational environment for analysis.

> ## Problems with creating environments with a script
> 1. The script assumes that you have access to certain software,
>   such as bash and FSL, on your system. This means you have to run
>   this on a unix-like system such as Linux or MacOS.
> 2. The script will install all other necessary software (e.g., python libraries and their dependencies) and
>   will not handle any conflicts with your existing software environment.
{: .callout}

Alternatives that reproduce environments with minimal software
dependencies are technologies like virtual machines ([VirtualBox](https://www.virtualbox.org/),
[VMWare](https://www.vmware.com/products/desktop-virtualization.html),
[NITRC CE](https://www.nitrc.org/projects/nitrc_es/)), containers
([Docker](https://www.docker.com/), [Singularity](http://singularity.lbl.gov/))
and installers (e.g., [Vagrant](https://www.vagrantup.com/),
[Packer](https://www.packer.io/)). These can be very useful to replicate existing
environments and therefore simplify the installation problem
significantly. However, at present some of these technologies are not
installed by default on computing clusters you may have access to.

[Lesson 3](../04-containers) covers container technologies and how to create, use, modify,
and reuse containers.

> ## Exercise:
>
>  What must a reproducible computational environment contain?
>
> > ## Solution
> >
> > A reproducible computational environment must contain all the necessary data
> > (any inputs or other internal software package data such as brain templates),
> > environment variables, and software necessary for carrying out the ascribed
> > computation. Ideally, such an environment itself should be reproducible.
> {: .solution}
{: .challenge}


<a name="element3" />

### Element 3: Running analysis and storing expected results and provenance

Once the environment is set up, one can execute the analysis workflow. Each
time the analysis is run the provenance of the workflow is captured and stored
using a [PROV](https://www.w3.org/TR/prov-dm/) model for workflows. All of
this happens inside a [single executable
script](https://github.com/ReproNim/simple_workflow/blob/master/run_demo_workflow.py).
The script `Simple_Prep.sh` uses [Nipype dataflows](http://nipy.org/nipype) to ensure a consistent representation of
the execution graph, itself a representation of the steps followed during this part of the analysis workflow.


> ## Using dataflow technologies for analysis instead of shell scripts
> There are many dataflow platforms out there. These typically enable a
> compact, abstract graph based representation of a dataflow, allowing
> for their reuse and consistent of execution. They also enable running the same
> dataflow in different computing environments and not requiring the
> user to keep track of complex data dependencies across nodes. While
> Nipype was used in this example, other brain imaging dataflow systems
> include Automated Analysis, PSOM, FASTR.
{: .callout}

Running the analysis is one part of reproducibility. It is important to
also capture the output necessary for scientific hypothesis testing or
exploration. In this example, the volumes of subcortical structures and
of the different brain tissue classes are extracted and stored in a JSON
document. A specific run of this workflow on a specific platform was
used to create the provenance document and the expected outputs data.
When another user runs this workflow, their output can be compared to
the expected output.

[Lesson 4](../05-dataflows) covers data flow technologies, specifically how to create
analysis pipelines and applications and capture provenance when running
these pipelines.

> ## Exercise:
>
>  What are some advantages of using dataflow technologies?
>
> > ## Solution
> >
> > 1. Compact and structured representation of analysis.
> > 2. Can be reused with minimal changes.
> > 3. Many dataflow tools can be used across different environments.
> > 4. The tools take care of data management.
> > 5. Many dataflow tools support distributed execution of steps.
> {: .solution}
{: .challenge}


<a name="element4" />

### Element 4: Checking output consistency using continuous integration

Once the data and environment are setup appropriately and the analysis
is run, it would be good to know if the same results, within some threshold, are
obtained when a dataset containing the similar data or a similar
workflow is used. These can be carried out using continuous integration
services, such as [Travis](https://travis-ci.org/), [CircleCI](https://circleci.com/),
[Jenkins](https://jenkins.io/), which allow for the execution of an analysis workflow and, automated comparison tests
as versions of data or software change.

> ## Continuous integration testing
> In typical brain imaging analyses there is a complex interaction
> between data, software, and scientific hypothesis testing results.
> Continuous integration services ensure that such results can be
> obtained consistently and provide a framework to evaluate when results
> diverge. While typically used for software testing, continuous
> integration has become essential for process management by creating a
> complete test cycle.
{: .callout}

In brain imaging, most published results rarely come with data and code
to allow for the retesting of the outcome when either the software version changes
or when new datasets are available. The intent of this simple workflow
example is to move the community towards such comprehensive data
preservation and testing integration.

[Lesson 5](../06-testing) covers how to use continuous integration services and
also how container technologies can be used to run your own integration testing.

> ## Exercise:
>
>  How does setting up continuous integration testing help with research?
>
> > ## Solution
> >
> > During the lifetime of a project, the data and software may change. Setting
> > up testing environments allow ensuring that changes in results can be
> > attributed to changes in data and software. Results that remain the same across
> > different software and similar data sets are more generalizable than results
> > that are specific to a particular dataset and a particular software
> > environment.
> {: .solution}
{: .challenge}


<a name="results" />

### Results from running the Simple Workflow example

It turns out that this Simple Workflow is not reproducible across different
versions of software and operating systems. [The observed inconsistencies (see Table 1)](https://f1000research.com/articles/6-124/)
point to issues of randomization and/or initialization within the algorithms that
are run. While its easy to detect deviation of execution in different
environments it is harder to determine the cause of the deviation. This
is where rich provenance capture can help establish where along an
execution graph an analysis diverged and help zero in on the possible
culprits.

[Lesson 6](../07-provenance) covers details of how provenance can be captured.

> ## Hands on exercise:
>
>  Submit only your json output and provenance files as a pull request?
>
> > ## Solution
> > 1. See an example [here](https://github.com/ReproNim/simple_workflow/tree/master/expected_output)
> > 2. Using your Unix skills (find, tar) extract only the json files keeping
> >    the directory structure intact.
> > 3. Add the provenance files (trig, provn)
> > 4. Fork the simple_workflow repo
> > 5. Add your outputs to a new folder under other_outputs.
> > 6. Commit the changes, push to your repo, and send a pull-request.
> >
> {: .solution}
>
>  What do the results indicate about neuroimaging software?
>
> > ## Solution
> >
> > Researchers have to be careful about variations coming from numerical software.
> > Software engineers have to test their software for numerical variation across
> > different operating systems and software environments. The only way to scale
> > this is using continuous integration approaches.
> {: .solution}
{: .challenge}
