# Building-intellegent-document-processing-solution

# Problem Statement : Building an end-to-end intelligent document processing solution using AWS

# Why am I building this project?
--> As organizations grow larger in size, so does the need for having better document processing. In industries such as healthcare, legal, insurance, and banking, the continuous influx of paper-based or PDF documents (like invoices, health charts, and insurance claims) have pushed businesses to consider evolving their document processing capabilities.


# Steps of solution :

--> How to build such application?
Main Steps :

1. Create S3 bucket
2. Create custom entity recognizer
3. Create human review workflow
4. Deploy Cloudformation stack


Full steps :

1. Creating S3 bucket : For storage
--> I have create new bucket named "textract-comprehend-a2i-data-18it003"
--> Create folder and name it. I have given it as "comprehend_data"
--> Upload two dataset files in csv format. I have uploaded "entity.csv" and "raw_txt.csv"
--> Copy bucket name as well as S3 URL of both dataset files as we will need it further for creating CloudFormation Stack.

2. Create IAM role named as TDA having policy S3 full access

3. Create custom entity recognizer : For training dataset 
--> Launch Amazon comprehend
--> Choose custom entity recognizer
--> Choose train recognizer
--> Enter recognizer name, entity type(parameter on which we want to train model)
--> I have entered recognizer name as "tda-custom-entity-recognizer". I have named entity type as "DEVICE".
--> Select using entity list and training docs and add entity location of S3 as well as training documents of S3 from Browse S3.
--> Select IAM role existing role(TDA).
--> Now click on train to train data model.
--> Wait till status changes to trained as firstly it will will be processing, then training and at last, trained.
--> Copy ARN of custom entity recognizer as it will be used further for creating CloudFormation stack.

4. For creating human review workflow, we first need to create workforce using Amazon Sagemaker. I will create private workforce as in vendors, we have to purchase from Marketplace and
   private workforce uses Amazon Cognito and we can manage team. 
--> Go to Amazon Sagemaker from console and select launch sagemaker studio.
--> From navigation panel, click on labeling workforce, this will redirect us to labeling workforce page.
--> Click on private from 3 options i.e. Amazon Mechanical Task, Private and vendor.
--> By choosing private, it will allow us to make private team and can send notification to all workers attached with it via email and worker get temporary password which can be changed by
    clicking link at the end of mail by using temporary username and password.
--> Now click on create private team.
--> Choose Invite workers by email, then enter email addresses list upto 50 seperated by commas in Email address box.As SNS is unabled, this will send notification to all 50 email addresses
    listed over here. I have entered 50 email addresses starting from 18it001@charusat.edu.in, 18it002@charusat.edu.in, ...., 18it050@charusat.edu.in .
--> Choose create private team and enter team name and select create a new Amazon Cognito user group under Addd workers section. This will create user pool in Cognito service with MFA 
    unabled and all necessary things like adding app client, adding users and group, adding SNS notification as email, creating seperate domain name which is available, etc.
--> Now, click on create private team. This will diplay Amazon Cognito user pool URL, App client URL(in which we can see Jobs after creating human workflow), labeling URL(in which user can
    sign in with temporary credentials and after changing password, it will show verified under Wokers section of team).
--> I have created private team named as "textract-comprehend-a2i-review-team".
--> You can now add workers to team as well as can invite them also.
--> All 50 email id listed above will receive an email notification saying "You're invited to work on labeling project." with username and password which are temporary and with one link to 
    login which is labeling link and user can login to get verified and changing of password is necessary as these are temporary and status under Workers section of team will show 
    FORCED_CHANGED untill and unless it is changed but after changing, it will show verified.
--> So, changing password is necessary.
--> After login into labeling protal, you will see page showing "Hello 'email'". Initially, it will be blanked as nothing is added and human workflow is remaining.
--> This portal is of Amazon A2I portal i.e. Augmented AI.

5. Creating Worker task templates : to customize interface and intructions that workers can see when working on given task.
   It is used when for Amazon Textract built-in task types, the conditions which your human loop will be called and workforce sent tasks and instruction which workforce receive is called 
   worker task template.
--> Go to Worker task templates from navigation panel of Amazon Sagemaker under Augmented AI section.
--> Enter template name. I have entered it as "custom-entity-review-template".
--> Under Template editor section, I have added following code :
    <script src="https://assets.crowd.aws/crowd-html-elements.js"></script>

    <crowd-entity-annotation
        name="crowd-entity-annotation"
        header="Highlight parts of the text below"
        labels="{{ task.input.labels | to_json | escape }}"
        initial-value="{{ task.input.initialValue }}"
        text="{{ task.input.originalText }}"
     >
     <full-instructions header="Named entity recognition instructions">
        <ol>
            <li><strong>Read</strong> the text carefully.</li>
            <li><strong>Highlight</strong> words, phrases, or sections of the text.</li>
            <li><strong>Choose</strong> the label that best matches what you have highlighted.</li>
            <li>To <strong>change</strong> a label, choose highlighted text and select a new label.</li>
            <li>To <strong>remove</strong> a label from highlighted text, choose the X next to the abbreviated label name on the highlighted text.</li>
            <li>You can select all of a previously highlighted text, but not a portion of it.</li>
        </ol>
      </full-instructions>

      <short-instructions>
         Highlight the custom entities that went missing.
      </short-instructions>

      </crowd-entity-annotation>

      <script>
        document.addEventListener('all-crowd-elements-ready', () => {
          document
            .querySelector('crowd-entity-annotation')
            .shadowRoot
            .querySelector('crowd-form')
            .form;
         });
      </script>

-->You can add any code you want.
--> Now click on create to create template. 

6. Creating human review workflow : for verification and predictions as this will allow easy prediction from Amazon Textract to review easily.
--> As this is workflow, this will trigger lambda function. It will be triggered when results of Amazon Comprehend will go to this workflow and again when this workflow's output will go 
    back to Amazon Comprehend. Lambda function will also be triggered when objects are modified in S3 bucket and this will help to Analyze job.
--> Go to human review workflow in Amazon Sagemaker under Augmented AI section.
--> Click on create human review workflow.
--> Under workflow settins, for Name, give unique name to S3 bucket. I had given name as "custom-entity-review-workflow" and for S3 bucket, I had given name as s3://textract-comprehend-a2i-
    data-18it003/temp
--> Choose create new IAM role.(You can select existing IAM role if you have existing IAM role of Sagemaker-Execution-Role with policy S3 bucket full access.
--> I don't have any Sagemaker role. so, I have created it and selected Specify S3 bucket and under that, I have typed name of my bucket i.e. "textract-comprehend-a2i-data-18it003"(primary 
    bucket).
--> By clicking on create, I have created IAM role and one message was displayed confirming about scuessfully creation of IAM role.
--> Under Task type, I have selected custom as I want custom entitiy to be recognized from 3 sections as Textract-Key-value pair selection, Rekognition-Image moderation and Custom.
--> Under Worker task template section, I had selected the template which I have created above i.e. "custom-entity-review-template".
--> Add description "Verify that all detects identities are correct and identify missing ones." in Task description.
--> Under Workers section, select Private in Woker types from 3 types i.e. Amazon Mechanical Turk, Private and Vendor and in that, will select private team created above i.e.
    "textract-comprehend-a2i-review-team".
--> Now click on create to create human review workflow.
--> Wait until status changed to active as it will be in intializing stage first, then at last, active.
--> Copy ARN of workflow as it will be later needed to create CloudFormation Stack.

7. Creating and Deploying CloudFormation stack.
--> Copy URL : https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://s3.amazonaws.com/aws-ml-blog/artifacts/Document-Analysis-Solution
    /Textract-Comprehend-A2I-Template.yaml and under stack name, enter name which you want to give. I have named it as TDA.
--> Paste ARN of custom entity recognizer which you have copied eariler in CustomEntityRecognizerARN.
--> Paste URL of training dataset that used for creating custom entity recognizer which you have copied eariler in CustomEntityTrainingDatasetS3URL(here URL of raw_txt.csv).
--> Paste URL of training entity list that used for creating custom entity recognizer which you have copied eariler in CustomEntityTrainingListS3URL(here URL of entity_list.csv).
--> Paste ARN of human review workflow which you have copied eariler in FlowDefinitionARN.
--> Enter bucket name in S3BucketName(here textract-comprehend-a2i-data-18it003).
--> Enter unique name in S3ComprehendBucketName as it will store temporarory output.Do not create this bucket as CloudFormation will create it. I have named it as "textract-comprehend-a2i-
    data-18it003-temp123".
--> Now after ticking all 3 checkboxes, click on create stack.
--> Wait until status changed to CREATE_COMPLETE after creating as it can go in ROLLBACK_COMPLETE also and delete IAM roles, etc. and in this stage, you only have to delete stack as you can not do 
    anything else.
--> After CREATE_COMPLETE, solution completes.
--> Cloudwatch is configured and hence, all the logs of activities you can find under log and also find bills of each service under bill section of Cloudwatch.

Testing Solution :

1.  Upload a file.
2. Verify Amazon Comprehend job status
3. Review worker portal
4. Verify the changes were recorded.


1. Uploading a file :
--> find scanned document with text written in it.
--> Go to Amazon S3 console, and click on primary bucket i.e. textract-comprehend-a2i-data-18it003.
--> Create folder and name it. I have given name as "input". 
--> Upload above text document inside that newly created folder.

2. Verify Amazon Comprehend job status :
--> Go to amazon Comprehend and go to Analysis jobs from navigation panel.
--> Wait until status changes to Completed as it will first show in progress.
--> You will see Documentname-Bucketname and status completed.

3. Reviewing worker portal :
--> Go to Amazon A2I worker portal.
--> The job will be waiting and job-description here is Human review task.
--> Select job and choose start working.
--> You will be redirected to review screen.
--> You will see some items are highlighted, that entities are here DEVICES i.e. parameter which you had passed to train model.
--> You will see not all devices are listed, so I have tag new entities which was missing and at last, all entities got highlighted(here all Devices).
--> When you are finished, click on submit.

4. Verify the changes were recorded :
--> After adding inputs into A2I console, HumanWorkflowCompleted Lambda function adds indentified entities to already existing file and stores it as seperate entity list in Amazon S3 bucket.
--> Go to your folder inside primary bucket, here "comprehend". 
--> You will see updated list of csv file.
--> I got that file named as "updated_entity_list.csv".
--> The NewEntityCheck Lambda function uses this file at the end of each day to compare against the original entity_list.csv file. If new entities are in the updated_entity_list.csv file, 
    the model is retrained and replaces the older custom entity recognition model.
--> This is machine learning model, so it will improve over time.
--> This allows the Amazon Comprehend custom entity recognition model to improve continuously by incorporating the feedback received from human reviewers through Amazon A2I. Over time, 
    this can reduce the need for reviewers and manual intervention by analyzing documents in a more intelligent and sophisticated manner.
