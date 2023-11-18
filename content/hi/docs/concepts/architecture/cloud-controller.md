---
title: क्लाउड कंट्रोलर प्रबंधक
content_type: concept
weight: 40
---

<!-- सवारानुमान -->

{{< feature-state state="beta" for_k8s_version="v1.11" >}}


क्लाउड इंफ्रास्ट्रक्चर तकनीक आपको सार्वजनिक, निजी, और हाइब्रिड क्लाउड पर Kubernetes चलाने की सुविधा प्रदान करती है। Kubernetes अटोमेटेड, एपीआई-निर्देशित इंफ्रास्ट्रक्चर की मानकों में मानता है जिसमें घटकों के बीच कोसन नहीं होती है।



क्लाउड-कंट्रोलर-मैनेजर एक प्लगइन मेकेनिज़म का उपयोग करके संरचित है जो विभिन्न क्लाउड प्रदाताओं को उनके प्लेटफ़ॉर्म को Kubernetes के साथ मेल करने की अनुमति देता है।

<!-- शरीर -->

## डिज़ाइन

![Kubernetes components](/images/docs/components-of-kubernetes.svg)

क्लाउड कंट्रोलर प्रबंधक नियंत्रण तबक में एक पुनरावृत्ति होती है (सामान्यत: ये पॉड में कंटेनर्स होते हैं)। प्रत्येक क्लाउड-कंट्रोलर-मैनेजर एक ही प्रक्रिया में कई {{< glossary_tooltip text="controllers" term_id="controller" >}}  को कार्रवाई करता है।

{{< note >}}
आप क्लाउड कंट्रोलर प्रबंधक को नियंत्रण तबक का {{< glossary_tooltip text="addon" term_id="addons" >}} बनाने के रूप में भी चला सकते हैं।
{{< /note >}}

## क्लाउड कंट्रोलर प्रबंधक कार्ये {#functions-of-the-ccm}

क्लाउड कंट्रोलर प्रबंधक के अंदर शामिल नियंत्रक हैं:

### नोड नियंत्रक

नोड नियंत्रक आपके क्लाउड इंफ्रास्ट्रक्चर में नए सर्वर बनाए जाने पर  {{< glossary_tooltip text="Node" term_id="node" >}} ऑब्जेक्ट्स को अपडेट करने के लिए जिम्मेदार है। नोड नियंत्रक आपके टेनेंसी में चल रहे होस्ट्स के बारे में जानकारी प्राप्त करता है जो क्लाउड प्रदाता के साथ होते हैं। नोड नियंत्रक निम्नलिखित कार्रवाई करता है:

1. नोड ऑब्जेक्ट को क्लाउड प्रदाता एपीआई से प्राप्त किए गए सर्वर की यूनिक पहचानी द्वारा अपडेट करें।
2. नोड ऑब्जेक्ट को क्लाउड-विशिष्ट जानकारी के साथ एनोटेट और लेबल करें, जैसे कि रीजन जिसमें नोड डिप्लॉय किया गया है और उन संसाधनों (सीपीयू, मेमोरी, आदि) के साथ जो इसमें उपलब्ध हैं।
3. नोड के होस्टनेम और नेटवर्क पते प्राप्त करें।
4. नोड की स्वास्थ्य की पुष्टि करें। यदि एक नोड उत्तरदाता नहीं होता है, तो यह नियंत्रक आपके क्लाउड प्रदाता की एपीआई से जाँच करता है कि क्या सर्वर निष्क्रिय / हटा दिया गया / समाप्त हो गया है।
   यदि नोड को क्लाउड से हटा दिया गया है, तो नियंत्रक नोड ऑब्जेक्ट को आपके Kubernetes
   क्लस्टर से हटा देता है।

कुछ क्लाउड प्रदाता अंमलण इसे नोड नियंत्रक और एक अलग नोड
लाइफसाइकिल नियंत्रक में विभाजित करते हैं।

### रूट नियंत्रक

रूट नियंत्रक यह जिम्मेदार है कि क्लाउड में रूट कैसे कॉन्फ़िगर करें
ऐसी ताकि आपके Kubernetes में विभिन्न नोडों पर कंटेनर
एक दूसरे से संवाद कर सकें।

क्लाउड प्रदाता के आधार पर, रूट नियंत्रक संभावना है कि यह ब्लॉक को आवंटित करेगा
पॉड नेटवर्क के लिए आईपी पतों के लिए।


### सेवा नियंत्रक

{{< glossary_tooltip text="सेवाएँ" term_id="service" >}} बादल अंगगत घटकों के साथ एकीकृत होती हैं, जैसे कि प्रबंधित लोड बैलेंसर, आईपी पते, नेटवर्क पैकेट फ़िल्टरिंग, और लक्ष्य स्वास्थ्य जाँच। सेवा नियंत्रक आपके
बादल प्रदाता के API के साथ परिक्रमा करता है ताकि जब आप एक सेवा संसाधित करते हैं जिसको इन घटकों की आवश्यकता हो, तो लोड बैलेंसर्स और अन्य बादल अंगगत घटकों को सेटअप कर सके।

## अधिकृतता

इस खंड में बताया गया है कि क्लाउड नियंत्रक प्रबंधक को अपने ऑपरेशन करने के लिए विभिन्न API ऑब्जेक्ट्स पर कौन-कौन सी पहुंच की आवश्यकता होती है।

### नोड नियंत्रक {#authorization-node-controller}

नोड नियंत्रक केवल नोड ऑब्जेक्ट्स के साथ काम करता है। इसे नोड ऑब्जेक्ट्स को पढ़ने और संशोधित करने के लिए पूर्ण पहुंच की आवश्यकता है।

`v1/Node`:

- get
- list
- create
- update
- patch
- watch
- delete
### रूट नियंत्रक {#authorization-route-controller}

रूट नियंत्रक नोड ऑब्जेक्ट के निर्माण को सुनता है और सही रूट कॉन्फ़िगर करता है। इसे नोड ऑब्जेक्ट्स के लिए Get पहुंच की आवश्यकता होती है।

`v1/Node`:

- get

### सेवा नियंत्रक {#authorization-service-controller}

सेवा नियंत्रक सेवा ऑब्जेक्ट **निर्माण**, **अपडेट**, और **हटाने** के घटनाओं का प्रतीक्षा करता है और फिर
उन सेवाओं के लिए उचित अंतक्षेत्र कॉन्फ़िगर करता है (EndpointSlices के लिए,
kube-controller-manager इन्हें आवश्यकता के हिसाब से प्रबंधित करता है)।

सेवाओं तक पहुंचने के लिए, इसे **सूची** और **देखो** पहुंच की आवश्यकता है। सेवाओं को अपडेट करने के लिए, इसे
**पैच** और **अपडेट** पहुंच की आवश्यकता है।

सेवाओं के लिए अंतक्षेत्र संसाधन सेटअप करने के लिए, इसे **निर्माण**, **सूची**, **प्राप्त करो**,
**देखो**, और **अपडेट** का पहुंच चाहिए।

`v1/Service`:

- सूची
- प्राप्त करो
- देखो
- पैच
- अपडेट
### अन्य {#authorization-miscellaneous}

क्लाउड नियंत्रक प्रबंधक के मौलिक के कार्यान्वयन के लिए स्थापना को ईवेंट ऑब्जेक्ट बनाने की पहुंच की आवश्यकता है,
और सुरक्षित प्रचालन सुनिश्चित करने के लिए, इसे सर्विस अकाउंट बनाने की पहुंच की आवश्यकता है।

`v1/Event`:

- create
- patch
- update

`v1/ServiceAccount`:

- create

क्लाउड नियंत्रक प्रबंधक के लिए {{< glossary_tooltip term_id="rbac" text="RBAC" >}} ClusterRole इस तरह दिखता है:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cloud-controller-manager
rules:
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - list
  - patch
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - serviceaccounts
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - persistentvolumes
  verbs:
  - get
  - list
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - endpoints
  verbs:
  - create
  - get
  - list
  - watch
  - update
```
## {{% heading "whatsnext" %}}

* [क्लाउड नियंत्रक प्रबंधक प्रशासन](/docs/tasks/administer-cluster/running-cloud-controller/#cloud-controller-manager)
  में क्लाउड नियंत्रक प्रबंधक को चलाने और प्रबंधित करने के निर्देश हैं।

* एक एचए कंट्रोल प्लेन को क्लाउड नियंत्रक प्रबंधक का उपयोग करने के लिए अपग्रेड करने के लिए, देखें
  [क्लाउड नियंत्रक प्रबंधक का उपयोग करने के लिए रिप्लिकेटेड कंट्रोल प्लेन को माइग्रेट करें](/docs/tasks/administer-cluster/controller-manager-leader-migration/).

* अपने खुद के क्लाउड नियंत्रक प्रबंधक को कैसे लागू करें, या किसी मौजूदा परियोजना को कैसे विस्तारित करें, इसके बारे में जानना चाहते हैं?

  - क्लाउड नियंत्रक प्रबंधक गो इंटरफेस का उपयोग करता है, विशेष रूप से, [`cloud.go`](https://github.com/kubernetes/cloud-provider/blob/release-1.21/cloud.go#L42-L69)
    में परिभाषित `CloudProvider` इंटरफेस को
    [kubernetes/cloud-provider](https://github.com/kubernetes/cloud-provider) से
    किसी भी क्लाउड से आवेगी अनुसंधान करने के लिए।
  - इस दस्तावेज़ में हाइलाइट किए गए साझा नियंत्रकों का अनुसंधान का कार्यान्वयन (नोड, रूट, और सेवा),
    और साझा क्लाउडप्रवाह इंटरफेस के साथ कुछ स्कैफोल्डिंग, कुबरनेटीज के कोर का हिस्सा है।
    क्लाउड प्रदाता के विशिष्ट अनुसंधान कोर के बाहर होते हैं और इस इंटरफेस को
    लागू करते हैं।
  - प्लगइन्स विकसित करने के बारे में अधिक जानकारी के लिए,
    [क्लाउड नियंत्रक प्रबंधक विकसित करना](/docs/tasks/administer-cluster/developing-cloud-controller-manager/) देखें।