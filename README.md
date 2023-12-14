Contingency Table: https://www.statology.org/contingency-table-python/
Chi-Square Critical value: https://www.statology.org/chi-square-critical-value-python/

The χ2 statistic tests the hypothesis that A and B are independent, that is, there is no
correlation between them. The test is based on a significance level, with (r − 1) × (c − 1)
degrees of freedom. We illustrate the use of this statistic in Example 3.1. If the hypothesis
can be rejected, then we say that A and B are statistically correlated.

The attribute with the maximum info gain is selected as the splitting attribute.
The attribute with the maximum gain ratio is selected as the splitting attribute.
The attribute with the minimum gini index is selected as the splitting attribute.

MultiLinear Regression: https://faun.pub/implementing-multiple-linear-regression-from-scratch-in-python-f5d84d4935bb




class Apriori(APIView):
    @method_decorator(csrf_exempt)
    def post(self, request, *args, **kwargs):
        if request.method == 'POST':
                
                file = request.FILES.get('file')
                itemsetFile = request.FILES.get('itemsetFile')
                support = float(request.POST.get('support'))
                confidence = float(request.POST.get('confidence'))
                ruleLength = int(request.POST.get('ruleLength'))

                # print(support)
                # print(confidence)




                # print(file)
                # print(itemsetFile)
                # print(support)
                # print(confidence)
                # print(ruleLength)

                itemsetFile_file = TextIOWrapper(itemsetFile.file, encoding='utf-8')
                itemsetFile_reader = csv.reader(itemsetFile_file, delimiter='\t')

                replacement_dict = {}

                for row in itemsetFile_reader:

                    # print(row[0].replace(' ','').split(',')[1:])
                    # for k in range(1,len(row[0].replace(' ','').split(',')[1:])) : 
                    s = str(row[0].replace(' ','').split(',')[1:]).replace('\\','')
                    replacement_dict[row[0].replace(' ','').split(',')[0]] = s
                

                csv_file = TextIOWrapper(file.file, encoding='utf-8')
                csv_reader = csv.reader(csv_file, delimiter='\t')

                n = 0

                count = {}



                for row in csv_reader:
                    n+=1
                    # print(row[0].replace(' ','').split(',')[1:])
                    # for k in range(1,len(row[0].replace(' ','').split(',')[1:])) : 
                    s = row[0].replace(' ','').split(',')[1:]
                    subsets = self.get_subsets(ruleLength, s)
                        
                        # print() 
                        # print() 
                        # print() 

                    for subset in subsets:
                        # print(subset)
                        if subset not in count:
                            count[subset] = 0
                        count[subset]+=1
                
                cnt = support*n
                print(cnt)
                lc = []
                
                for key in count.keys():
                    print(key, count[key])
                    if count[key] >= cnt :
                        lc.append(key)

                association_rules = []
                
                for key in lc:
                    # print("k = ", key)

                    rules = self.generate_rules(key)

                    for rule, complementary_items in rules.items():
                        if count[key]/count[rule] >= confidence:

                            print(f'{tuple(replacement_dict.get(item, item) for item in rule)} ==> {tuple(replacement_dict.get(item, item) for item in complementary_items)}')
                            
                            contengency_table = [
                                [count[rule], count[key]-count[rule]],
                                [count[complementary_items]-count[rule], n-count[rule]]
                            ]

                            expected_table = [
                                [(contengency_table[0][0]+contengency_table[0][1])*(contengency_table[0][0]+contengency_table[1][0])/(contengency_table[0][0]+contengency_table[0][1]+contengency_table[1][0]+contengency_table[1][1]), (contengency_table[0][0]+contengency_table[0][1])*(contengency_table[0][1]+contengency_table[1][1])/(contengency_table[0][0]+contengency_table[0][1]+contengency_table[1][0]+contengency_table[1][1])],
                                [(contengency_table[0][0]+contengency_table[1][0])*(contengency_table[1][0]+contengency_table[1][1])/(contengency_table[0][0]+contengency_table[0][1]+contengency_table[1][0]+contengency_table[1][1]), (contengency_table[1][0]+contengency_table[1][1])*(contengency_table[0][1]+contengency_table[1][1])/(contengency_table[0][0]+contengency_table[0][1]+contengency_table[1][0]+contengency_table[1][1])]
                            ]

                            chi2test = (((contengency_table[0][0]-expected_table[0][0])**2)/expected_table[0][0])+(((contengency_table[0][1]-expected_table[0][1])**2)/expected_table[0][1])+(((contengency_table[1][0]-expected_table[1][0])**2)/expected_table[1][0])+(((contengency_table[1][1]-expected_table[1][1])**2)/expected_table[1][1])

                            conf1 = count[key]/count[rule]
                            conf2 = count[key]/count[complementary_items]



                            association_rules.append({
                                'rule': f'{tuple(replacement_dict.get(item, item) for item in rule)} ==> {tuple(replacement_dict.get(item, item) for item in complementary_items)}', 
                                'confidence': round(count[key]/count[rule], 2), 
                                'lift': round((count[key]*n)/(count[rule]*count[complementary_items]), 2),
                                'chi2test': round(chi2test, 2),
                                'all_confidence': round(count[key]/max(count[rule], count[complementary_items]), 2),
                                'max_confidence': round(max(conf1, conf2), 2),
                                'kulczynski': round((conf1+conf2)/2, 2),
                                'cosine': round(count[rule]/math.sqrt(count[rule]*count[complementary_items]), 2)
                                })
                    

                
                # print(count[(0,1,)])
                         


                # for rule in association_rules:
                #     print(rule)
                
                
                return JsonResponse({"data": association_rules})
            


    def get_subsets(self, ruleLength, input_list):
        n = len(input_list)
        n = max(n, ruleLength)
        subsets = []
        for r in range(1, ruleLength + 1):
           
            subset_combinations = combinations(input_list, r)
            subsets.extend(subset_combinations)
        return subsets
    
    def generate_rules(self, items):
        rules = {}
        n = len(items)

        for r in range(1, n):
            for combination in combinations(items, r):
                complementary_items = tuple(item for item in items if item not in combination)
                rules[combination] = complementary_items

        return rules


