---
title: Calculating advanced aggregations with Elastic Search and Django
date: 2019-07-28 09:00:01 Z
categories:
  - python
  - django
  - elastic search
layout: post
summary: There are several packages that depend on each other and can be used to set up elastic search in a Django project. Here I mention the relationships between them and how to extend them to calculate complex aggregations.
---

[Refine.bio](https://www.refine.bio) is a Django project that uses elastic search to power a webpage, that allows people to search transcriptome data in several experiments. In the beginning of the project we started using regular postgresql queries until the performance got shitty.

It's relatively straightforward to use [django-elasticsearch-dsl-drf](https://github.com/barseghyanartur/django-elasticsearch-dsl-drf) to index a set of objects in elastic search and serve them through an api endpoint. However things get more complicated if you need something less conventional. For example, in our case we allow people to search for experiments, each one associated with multiple samples through a many-to-many relationship. We wanted to [count the number of unique Samples](https://github.com/AlexsLemonade/refinebio/issues/1145) for each one of the filters returned. Something that quickly pops to the eye, are elastic search's [aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html) to perform computations on the indexed documents.

[The final solution](https://github.com/AlexsLemonade/refinebio/pull/1416) consisted on indexing all sample ids for each experiment and then using the [cardinality aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-cardinality-aggregation.html) to **estimate** the number of unique samples for each filter category. The real challenge came down to knowing how to implement that into the project, since there're several packages built on top of each other that each adds a different layer of abstraction.

The top level package [django-elasticsearch-dsl-drf](https://github.com/barseghyanartur/django-elasticsearch-dsl-drf) "provides views, serializers, filter backends, pagination and other handy add-ons", basically everything you need to integrate elastic search with [Django's Rest Framework](https://www.django-rest-framework.org/). It's built on top of [django-elasticsearch-dsl](https://github.com/sabricot/django-elasticsearch-dsl) and requires the document object to be defined by it.

Elastic search document objects contains specifications of how a given model should be serialized and indexed. [django-elasticsearch-dsl](https://github.com/sabricot/django-elasticsearch-dsl) provides helpers for this, however it's only a "thin wrapper" around [elasticsearch-dsl-py](https://github.com/elastic/elasticsearch-dsl-py), a "high-level library whose aim is to help with writing and running queries against Elasticsearch".

All those packages depend on the official low-level client [elasticsearch-py](https://github.com/elastic/elasticsearch-py). This package is completely abstracted by the others, and I didn't need to use any objects from it. Mentioning here for completeness.

![packages and their dependencies](/assets/img/2019-07-19-elastic-search-packages.png)

## The implementation

As mentioned before the implementation consisted of two steps. First getting the Sample ids indexed for each document

```py
class Experiment(models.Model):
  # ...

  @property
  def downloadable_samples():
    # ...
    return sample_ids
```

And the field on the `ExperimentDocument` so that it gets indexed.

```py
class ExperimentDocument(DocType):
  class Meta:
    model = Experiment

  # ... other fields

  downloadable_samples = fields.ListField(
      fields.KeywordField()
  )
```

`django-elasticsearch-dsl-drf` provides a way to add [nested aggregations/facets](https://django-elasticsearch-dsl-drf.readthedocs.io/en/0.18/nested_fields_usage_examples.html#nested-aggregations-facets). However this works when you need to add additional computations to the set of facets, but in the case of [refine.bio](https://www.refine.bio) we needed to add additional computations to the set of filters that were being returned by `FacetedSearchFilterBackend`.

I couldn't find any solution documented for this, so I looked into the code of `FacetedSearchFilterBackend` and noticed that the method `aggregate` could be extended to add additional aggregations for each bucket.

```py
class FacetedSearchFilterBackendExtended(FacetedSearchFilterBackend):
    def aggregate(self, request, queryset, view):
      facets = self.construct_facets(request, view)
      for field, facet in iteritems(facets):
          agg = facet['facet'].get_aggregation()
          queryset.aggs.bucket(field, agg)\
              .metric('total_samples', 'cardinality', field='downloadable_samples', precision_threshold=40000)
      return queryset
```

## Conclusion

There are multiple packages built on top of each other that are relevant to get Elastic Search working on a Django project. It's important to know what each one does to find the appropriate documentation, although in some cases there's no way around checking the code directly.
