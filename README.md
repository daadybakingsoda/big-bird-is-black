include <string>
#include <unordered_set>
#include <vector>
#include <memory>
#include <limits>

struct symptom;
using symptom_t = std::shared_ptr<symptom const>;
using symptom_list = std::vector<symptom_t>;

struct symptom {
  std::string name;
  double cost;
  double infectivity;
  std::unordered_set<symptom_t> next_symptoms;
};

auto estimateMostEfficientRoutes(
    double sum,
    double current_infectivity,
    symptom_list const& r0,
    std::unordered_set<symptom_t> const& symptoms)
      -> std::vector<std::pair<double, symptom_list>> {
  if (symptoms.empty()) {
    return {{sum, r0}};
  }

  std::vector<std::pair<double, symptom_list>> results;
  double max_sum = std::numeric_limits<double>::lowest();
  for (auto& s : symptoms) {
    symptom_list r1 = r0;
    r1.push_back(s);

    std::unordered_set<symptom_t> remaining_symptoms = symptoms;
    for (auto& s_ : s->next_symptoms) {
      remaining_symptoms.insert(s_);
    }
    for (auto& s_ : r1) {
      remaining_symptoms.erase(s_);
    }

    std::vector<std::pair<double, symptom_list>> rs =
        estimateMostEfficientRoutes(
            sum + s->cost * current_infectivity,
            current_infectivity + s->infectivity,
            r1, remaining_symptoms);
    for (auto& r : rs) {
      if (max_sum < r.first) {
        results.clear();
        max_sum = r.first;
        results.push_back(std::move(r));
      } else if (max_sum == r.first) {
        results.push_back(std::move(r));
      }
    }
  }
  return results;
}

std::unordered_set<symptom_t> create_symptoms() {
  auto pulmonary_edema = std::make_shared<symptom>();
  pulmonary_edema->name = "肺水腫";
  pulmonary_edema->cost = 7;
  pulmonary_edema->infectivity = 5;

  auto nausea = std::make_shared<symptom>();
  nausea->name = "吐き気";
  nausea->cost = 2;
  nausea->infectivity = 1;

  {
    auto vomiting = std::make_shared<symptom>();
    vomiting->name = "嘔吐";
    vomiting->cost = 3;
    vomiting->infectivity = 3;
    nausea->next_symptoms.insert(vomiting);

    vomiting->next_symptoms.insert(pulmonary_edema);

    auto diarrhoea = std::make_shared<symptom>();
    diarrhoea->name = "下痢";
    diarrhoea->cost = 6;
    diarrhoea->infectivity = 6;
    vomiting->next_symptoms.insert(diarrhoea);
    pulmonary_edema->next_symptoms.insert(diarrhoea);
  }

  auto coughing = std::make_shared<symptom>();
  coughing->name = "咳";
  coughing->cost = 4;
  coughing->infectivity = 3;

  {
    auto sneezing = std::make_shared<symptom>();
    sneezing->name = "くしゃみ";
    sneezing->cost = 5;
    sneezing->infectivity = 5;
    coughing->next_symptoms.insert(sneezing);

    auto pneumonia = std::make_shared<symptom>();
    pneumonia->name = "肺炎";
    pneumonia->cost = 3;
    pneumonia->infectivity = 3;
    coughing->next_symptoms.insert(pneumonia);

    pneumonia->next_symptoms.insert(pulmonary_edema);
  }

  auto rash = std::make_shared<symptom>();
  rash->name = "発疹";
  rash->cost = 3;
  rash->infectivity = 2;

  {
    auto sweating = std::make_shared<symptom>();
    sweating->name = "発汗";
    sweating->cost = 3;
    sweating->infectivity = 2;
    rash->next_symptoms.insert(sweating);

    auto skin_lesions = std::make_shared<symptom>();
    skin_lesions->name = "皮膚障害";
    skin_lesions->cost = 8;
    skin_lesions->infectivity = 11;
    sweating->next_symptoms.insert(skin_lesions);
  }

  auto cysts = std::make_shared<symptom>();
  cysts->name = "嚢胞";
  cysts->cost = 2;
  cysts->infectivity = 2;

  {
    auto abscesses = std::make_shared<symptom>();
    abscesses->name = "膿瘍";
    abscesses->cost = 2;
    abscesses->infectivity = 4;
    cysts->next_symptoms.insert(abscesses);
  }

  auto anaemia = std::make_shared<symptom>();
  anaemia->name = "貧血";
  anaemia->cost = 2;
  anaemia->infectivity = 1;

  {
    auto haemophilia = std::make_shared<symptom>();
    haemophilia->name = "血友病";
    haemophilia->cost = 3;
    haemophilia->infectivity = 4;
    anaemia->next_symptoms.insert(haemophilia);
  }

  return {nausea, coughing, rash, cysts, anaemia};
}

#include <iostream>

int main() {
  auto results = estimateMostEfficientRoutes(0, 4, {}, create_symptoms());
  for (auto& result : results) {
    std::cout << result.first << " ";
    for (auto& s : result.second) {
      std::cout << s->name << " ";
    }
    std::cout << std::endl;
  }
}
