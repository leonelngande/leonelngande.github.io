---
layout: post
hidden: false
title: How YAML Helped Me Format and Process Text Data From Documents Much Faster
tags:
  - YAML
  - Text Processing
---
I'll start by saying YAML is awesome! It's like that thing you've been looking for but never knew existed, at least for my use case.

#### Background

I'm working on a site which requires extraction of text data (past questions) from PDF and Word documents, even websites. This data has to be stored in an intermediate document using a predefined format, so it can later be processed and put into database tables.

A typical question paper looks like this PDF document: 

ADD PDF DOCUMENT HERE

And the database schema looks something like this:

![Cross section of database schema](/images/uploads/database-sample.png "Cross section of database schema")

Summarily,

* `Exam Providers` are exam issuing bodies (like the Cameroon General Certificate of Education Board). Each exam provider can issue many exams, e.g the Cameroon GCE Advanced Level and Ordinary Level examinations.
* `Exam`s may or may not belong to exam providers, and can have many `Courses`. Courses here represent the various subjects in an exam, e.g Biology, Chemistry, Computer Science, and Physics subjects in the Cameroon GCE Advanced Level Examination.

  `Courses` can have many `Exam Session`s, which represent the various times the exam was written (e.g in 2017, 2018, and 2019).

  `Exam Session`s can have many `Exam Section`s, which represent various sections in the exam questions paper, e.g Section A, Section B, and Section C in the above document.

  E`xam Section`s can contain sub `Exam Section`s. They can have many `Exam Section Question`s as well as a `Question Type`.

#### Initial Solution to storing this data for later processing.

*Note*: I am using the [Laravel PHP](https://laravel.com) framework to build this layer (API) of the application, so there's a model class for each of the tables in the above database schema.

My initial solution was copying the questions into `.txt` files, using specific keys to identify entities, and tab indentation for hierarchy. Here's a sample using the above question paper:

```
Provider: Cameroon GCE Board
Exam: Advanced Level
Course: Economics
Session: June 2015
Duration: Three hours


Section: PAPER 2



Section: SECTION A

1. “Despite the numerous advantages of the market economic system some countries still prefer the planned economic system”. Discuss. (20 marks)

2.
    (a) Distinguish between a change in supply and a change in quantity supplied. (10 marks)
    (b) What factors will lead to an increase in supply? (10 marks)

3. Explain
    (a) the similarities and
    (b) the differences between Perfect competition and Monopolistic competition, (10 + 10 marks)

4.
    (a) What do you understand by piece rate and time rate as methods of reward for labour? (6 marks)
    (b) What are the advantages and disadvantages of the piece rate method? (8 + 6 marks)



Section: SECTION B

5. Why might an increase in money national income not necessarily lead to an increase in the standards of living in a country? (20 marks)

6. The Franc CFA is money used in Cameroon. Why is it considered as good money? (20 marks)

7. Explain the principles of taxation and give reasons for the imposition of a tax. (10 + 10 marks)

8.
    (a) What constitutes the current account of the balance of payments? (8 marks)
    (b) How may favourable terms of trade improve upon a country's balance of payments situation? (12 marks)



Section: SECTION C

9. What factors will a manager take into consideration when setting the price of a product? (20 marks)

10. Examine the benefits and costs of multinational companies to the economy of Cameroon. (20 marks)

```

The idea was to read each line of the file, identify what it represents by scanning for specific keys in the line, as well as what it falls under by counting the number of tabs in relation to it's parent.

I quickly ruled this out given how much work it'll entail.

I then looked into a couple other options including Parsing Expression [Grammars](https://www.viget.com/articles/write-you-a-parser-for-fun-and-win/) ([PEGs](https://www.gnu.org/software/guile/manual/html_node/PEG-Parsing.html)) and [YAML](https://www.youtube.com/watch?v=cdLNKUoMc6c). After reading [this article](https://www.babeheim.com/2017/02/12/hierarchical-data-in-plain-text.html), I immediately knew YAML is what I was looking for. I no longer had to worry about manually scanning each line for specific keys, a YAML parser can do that automatically! Even better, it puts all sub items of each key in an easily traversable array.

Here's the yaml structure I ended up with for the above exam questions file:

```yaml
header:
    provider: Cameroon GCE Board
    exam: Advanced Level
    course: Economics
    session: June 2019
    duration: Three hours


sections:
    PAPER 2:
        sections:

            SECTION A:
                questions:
                    1:
                        questions:
                            (a): With the help of a diagram, explain what is meant by the optimum population seize of a country. (8 marks)
                            (b): What are the negative consequences of net emigration of people between the ages of 18 and 60 years on an economy? (12 mark)
                    2:
                        questions:
                            (a): Why might the price of cocoa fluctuate more than the prices of computers? (8 marks)
                            (b): What factors can lead to an increase in the supply of cocoa? (12 marks)
                    3:
                        questions:
                            (a): What are profits maximized at the output level where MC=MR? (12 marks)
                            (b): What others goals can firms seek to achieve apart from profit maximization? (8 marks)
                    4:
                        questions:
                            (a): Explain the loanable funds theory on interest rate determination. (10 marks)
                            (b): What are the factors that might likely cause a rise in interest rates in an economy? (10 marks)

            SECTION B:
                questions:
                    5:
                        questions:
                            (a): How is national income measured using the income method? (8 marks)
                            (b): Advance reasons for the rise in the national income of Cameroon over the years. (12 marks)
                    6:
                        questions:
                            (a):
                                question: What do you understand by
                                questions:
                                    (i): Demand pull inflation?
                                    (ii): Cost push inflation? (8 marks)
                            (b): What measures can be taken to remedy the negative effects of inflation in an economy? (12 marks)
                    7:
                        questions:
                            (a): Distinguish between tax incidence and tax burden. (6 marks)
                            (b): Why are most countries today shifting their tax based ways from direct taxes to indirect taxes? (14 marks)
                    8:
                        questions:
                            (a): How is a Customs Union different from a Free Trade Area? (8 marks)
                            (b): What are the likely benefits to Cameroon of being a member of CEMAC? (12 marks)
```

With the amount of flexibility it gives, I can now format the questions using a structure similar to the database schema, making the resulting data import script less of a challenge to write.

Using Symfony's [Yaml Component](https://symfony.com/doc/current/components/yaml.html#what-is-it) as my Yaml parser, I then wrote the following Laravel Artisan Command for processing the yaml files.

Hey this code was written just yesterday, so don't roast me if you see something's off 🙃, it does what it's suppose to do.

```php
<?php

namespace App\Console\Commands;

use App\Models\Course;
use App\Models\Exam;
use App\Models\ExamProvider;
use App\Models\ExamSection;
use App\Models\ExamSectionQuestion;
use App\Models\ExamSession;
use App\Models\Question;
use App\Models\QuestionType;
use Exception;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Storage;
use Symfony\Component\Yaml\Yaml;
use Throwable;

class ImportYamlExamDocument extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'exam:import-yaml';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Import an exam from a .yaml document.';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $files = Storage::files('questions/yaml');
        $selectedYamlFile = $this->choice('Select file', $files);

        $examArray = Yaml::parseFile(storage_path("app/$selectedYamlFile"));
        $examObject = (object) $examArray;

        try {
            DB::transaction(function () use ($examObject) {

                /** @var ExamProvider $examProvider */
                $examProvider = ExamProvider::firstOrCreate([
                    'name' => trim($examObject->header['provider'])
                ]);
                $this->info("Exam provider created: {$examProvider->name}");

                /** @var Exam $exam */
                $exam = Exam::firstOrCreate([
                    'name' => trim($examObject->header['exam']),
                    'exam_provider_id' => $examProvider->id,
                ]);
                $this->info("Exam created: {$exam->name}");

                /** @var Course $course */
                $course = Course::firstOrCreate([
                    'name' => trim($examObject->header['course']),
                    'exam_id' => $exam->id,
                ]);
                $this->info("Course created: $course->name");

                /** @var ExamSession $examSession */
                $examSession = ExamSession::firstOrCreate([
                    'name' => trim($examObject->header['session']),
                    'course_id' => $course->id,
                ]);
                $this->info("Exam session created: $examSession->name");

                /**
                 * Get and set Duration
                 */
                try {
                    $examSession->duration = trim($examObject->header['duration']);
                    $examSession->save();
                    $this->info("Exam session duration saved: $examSession->duration");
                } catch (Exception $e) {
                    $this->error($e->getMessage());
                }

                /**
                 * Print error and exit if no exam section is found in document
                 * There must be at least one exam section
                 */
                if (empty( $examObject->sections) ) {
                    throw new Exception("There must be at least one section.");
                }

                // $this->info(dd($sectionsWithIndexes));
                foreach ($examObject->sections as $sectionTitle => $sectionData) {

                    if (! is_array($sectionData)) {
                        throw new Exception("Non array value passed in as exam section data.");
                    }

                    $this->createExamSection($sectionTitle, $sectionData, $examSession);
                }

                if (! $this->confirm('Save to database?')) {
                    throw new Exception("Save to database cancelled.");
                }
            });
        } catch (Throwable $e) {
            $this->error($e->getMessage());
        }
    }

    /**
     * @param string $sectionTitle
     * @param array $sectionData
     * @param ExamSession $examSession
     * @param int|null $parentId
     * @throws Exception
     */
    public function createExamSection(string $sectionTitle, array $sectionData, ExamSession $examSession, int $parentId = null)
    {
        $section = ExamSection::firstOrCreate([
            'title' => trim($sectionTitle),
            'parent_id' => $parentId,
            'exam_session_id' => $examSession->id,
        ]);
        $this->info("Exam section created: {$section->title}");

        // warn if no question(s) found for section
        if (! isset($sectionData['questions']) ) {
            $this->warn("No questions found for section $section->title");
        } else {
            foreach($sectionData['questions'] as $questionGuiOrder => $questionValue) {
                $this->createExamSectionQuestion($section->id, $questionGuiOrder, $questionValue);
            }
        }

        if ( isset($sectionData['sections']) ) {
            foreach ($sectionData['sections'] as $childSectionTitle => $childSectionData) {
                /** @note Recursive call */
                $this->createExamSection($childSectionTitle, $childSectionData, $examSession, $section->id);
            }
        }
    }

    /**
     * @param int $examSectionId
     * @param $questionGuiOrder
     * @param array|string $questionData
     * @param int|null $parentId
     * @throws Exception
     */
    public function createExamSectionQuestion(int $examSectionId, $questionGuiOrder, $questionData, int $parentId = null)
    {
        if ( is_string($questionData) ) {
            $createdQuestion = $this->createQuestion($questionData);
        }
        elseif ( is_array($questionData) ) {
            // Create an empty question to be assigned to the exam section question
            $createdQuestion = $this->createQuestion(isset($questionData['question']) ? $questionData['question'] : '');
        }
        else {
            throw new Exception("Non string/array value passed as question data: " . gettype($questionData));
        }

        $questionType = QuestionType::firstOrCreate([
            'code' => 'open_response',
            // 'name' => 'Open response',
        ]);

        /** @var ExamSectionQuestion $sectionQuestion */
        $sectionQuestion = ExamSectionQuestion::firstOrCreate([
            'question_id' => $createdQuestion->id,
            'gui_order' => $questionGuiOrder,
            'exam_section_question_id' => $parentId,
            'question_type_id' => $questionType->id,
            'exam_section_id' => $examSectionId,
        ]);
        $this->info("Exam section question created: {$sectionQuestion->gui_order}");

        if (is_array($questionData)) {
            foreach ($questionData['questions'] as $childQuestionGuiOrder => $childQuestionValue) {
                /** @note Recursive call */
                $this->createExamSectionQuestion($examSectionId, $childQuestionGuiOrder, $childQuestionValue, $sectionQuestion->id);
            }
        }
    }

    public function createQuestion(string $question): Question
    {
        /** @var Question */
        $question = Question::firstOrCreate([
            'question' => trim($question),
        ]);

        return $question;
    }
}

```

#### Final thoughts.

Have you done something similar before? If yes i'd love to hear your approach. Please leave a comment or reach out to me on [Twitter](https://twitter.com/leonelngande).
