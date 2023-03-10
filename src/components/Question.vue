<template>
  <div v-if="question.isEnabled" :class="
  'question ' +
  'question-' +
  question.type +
  (question.isInvalid ? ' invalid' : '')
">
    <h2>
      {{
    (question.prefix ? question.prefix + ': ' : '') +
    question.label[language]
}}
    </h2>
    <!-- CHOICE Question -->
    <ul v-if="question.type === 'choice'">
      <li v-for="answer in question.answerOptions" :key="question.id + '-' + answer.code"
        @click="onAnswer(question, answer)">
        <input :type="question.allowsMultipleAnswers ? 'checkbox' : 'radio'" :checked="isSelected(question, answer)"
          :name="question.id" :id="answer.code.toString()" />
        <label for="answer.code.toString()">{{
    answer.answer[language]
}}</label>
      </li>
    </ul>
    <!-- STRING | DECIMAL Question -->
    <q-input v-if="question.type === 'string' || question.type === 'decimal'" v-model="value"
      @change="updateValue(value, question.type)" />

    <!-- INTEGER Question -->
    <q-input v-if="question.type === 'integer'" v-model="value" type="number"
      @change="updateValue(value, question.type)" />

    <!-- DATE Question -->
    <q-input v-if="question.type === 'date'" v-model="value" type="date" @change="updateValue(value, question.type)" />

    <!-- TEXT Question -->
    <textarea v-if="question.type === 'text'" v-model="value" @change="updateValue(value, question.type)">
    </textarea>

    <!-- BOOLEAN Question -->
    <input v-if="question.type === 'boolean'" type="checkbox" v-model="booleanValue"
      @change="updateValue(booleanValue, question.type)" />

    <!-- SUB Questions-->
    <div v-if="question.subItems && question.subItems.length > 0">
      <QuestionComponent v-for="subquestion of question.subItems" :question="subquestion" :key="subquestion.id"
        :isSelected="isSelected" :onAnswer="onAnswer" :language="language" />
    </div>
  </div>
</template>

<script lang="ts">
import { QuestionnaireItemType } from '@i4mi/fhir_r4';
import { defineComponent } from 'vue';
import { IAnswerOption, IQuestion } from '@i4mi/fhir_questionnaire';

export default defineComponent({
  name: 'QuestionComponent',
  data() {
    return {
      value: '' as string | number,
      booleanValue: false,
    };
  },
  props: {
    question: {
      type: Object, // as PropType<IQuestion>,
      required: true,
    },
    onAnswer: {
      type: Function, //as PropType<(q: IQuestion, a: IAnswerOption) => void>,
      required: true,
    },
    isSelected: {
      type: Function, // as PropType<(q: IQuestion, a: IAnswerOption) => boolean>,
      required: true,
    },
    language: {
      type: String,
      required: true,
    },
  },
  beforeMount() {
    const question = this.$props.question as IQuestion;
    if (question.type === QuestionnaireItemType.STRING && question.selectedAnswers.length > 0) {
      this.value = question.selectedAnswers[0] as unknown as string;
    }

  },
  methods: {
    updateValue(value: string | number | boolean, type: QuestionnaireItemType) {
      const answer: IAnswerOption = {
        answer: {},
        code: {},
      };
      switch (type) {
        case QuestionnaireItemType.STRING:
        case QuestionnaireItemType.TEXT: {
          answer.answer[this.language] = value as string;
          answer.code.valueString = value as string;
          break;
        }
        case QuestionnaireItemType.DECIMAL: {
          answer.answer[this.language] = value as string;
          answer.code.valueDecimal = value as number;
          break;
        }
        case QuestionnaireItemType.INTEGER: {
          answer.answer[this.language] = value as string;
          answer.code.valueInteger = value as number;
          break;
        }
        case QuestionnaireItemType.BOOLEAN: {
          answer.answer[this.language] = value as string;
          answer.code.valueBoolean = value as boolean;
          break;
        }
        case QuestionnaireItemType.DATE: {
          const date = new Date(value as string);
          answer.answer[this.language] = date.toLocaleTimeString();
          answer.code.valueDate = date.toISOString();
          break;
        }
      }
      this.onAnswer(this.question, answer);
    },
  },
});
</script>

<style scoped>
.question {
  border-left: solid 1px white;
  border-radius: 1px;
  padding: 0.3em 0.5em;
  margin-bottom: 0.5em;
}

.question-display {
  border-left: unset;
}

.question h2 {
  font-size: 1em;
  line-height: 1em;
  font-weight: bold;
  margin: 0;
}

.question.invalid {
  border-left: solid 3px #d73e2e;
  border-radius: 3px;
  background-color: #533;
}

.question-display h2 {
  font-weight: normal;
}

.question-choice ul {
  list-style: none;
  padding-left: 0.5em;
}

.question-choice li {
  cursor: pointer;
  margin-bottom: 0.5em;
}

.question-choice input {
  cursor: pointer;
}

.question-choice label {
  margin-left: 0.5em;
  cursor: pointer;
}
</style>
