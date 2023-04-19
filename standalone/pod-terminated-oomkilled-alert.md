curl -X PUT "https://elastic:xFJleiER1v67t2xq5RuLdrna@kubeconeu.es.europe-west1.gcp.cloud.es.io/_watcher/watch/Pod-Terminated-OOMKilled?pretty" -k -H 'Content-Type: application/json' -d'
{
  "trigger": {
    "schedule": {
      "interval": "1m"
    }
  },
  "input": {
    "search": {
      "request": {
        "search_type": "query_then_fetch",
        "indices": [
          "*"
        ],
        "rest_total_hits_as_int": true,
        "body": {
          "size": 0,
          "query": {
            "bool": {
              "must": [
                {
                  "range": {
                    "@timestamp": {
                      "gte": "now-1m",
                      "lte": "now",
                      "format": "strict_date_optional_time"
                    }
                  }
                },
                {
                  "bool": {
                    "must": [
                      {
                        "query_string": {
                          "query": "data_stream.dataset: kubernetes.state_container",
                          "analyze_wildcard": true
                        }
                      },
                      {
                        "exists": {
                          "field": "kubernetes.container.status.last_terminated_reason"
                        }
                      },
                      {
                        "query_string": {
                          "query": "kubernetes.container.status.last_terminated_reason: OOMKilled",
                          "analyze_wildcard": true
                        }
                      }
                    ],
                    "filter": [],
                    "should": [],
                    "must_not": []
                  }
                }
              ],
              "filter": [],
              "should": [],
              "must_not": []
            }
          },
          "aggs": {
            "pods": {
              "terms": {
                "field": "kubernetes.pod.name",
                "order": {
                  "_key": "asc"
                }
              }
            }
          }
        }
      }
    }
  },
  "condition": {
    "array_compare": {
      "ctx.payload.aggregations.pods.buckets": {
        "path": "doc_count",
        "gte": {
          "value": 1,
          "quantifier": "some"
        }
      }
    }
  },
  "actions": {
    "ping_slack": {
      "foreach": "ctx.payload.aggregations.pods.buckets",
      "max_iterations": 500,
      "webhook": {
        "method": "POST",
        "url": "https://hooks.slack.com/services/somesome/somesome/plokijuhyg",
        "body": "{\"channel\": \"#k8s-alerts\", \"username\": \"k8s-cluster-alerting\", \"text\": \"Pod {{ctx.payload.key}} was terminated with status OOMKilled.\"}"
      }
    }
  },
  "metadata": {
    "xpack": {
      "type": "json"
    },
    "name": "Pod Terminated OOMKilled"
  }
}
'