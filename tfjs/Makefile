MODEL = g104-b6c96-s97778688-d23397744
CHECKPOINT = ../models/$(MODEL)/saved_model/variables/variables
TMP_FOLDER = tmp
MODEL_GRAPH = $(TMP_FOLDER)/graph.pbtxt
FROZEN_MODEL = $(TMP_FOLDER)/frozen_model.pb
OPTIMIZED_MODEL = $(TMP_FOLDER)/optimized.pb
SAVED_MODEL = saved_model/saved_model.pb

all: web_model

$(MODEL_GRAPH): Makefile
	mkdir -p tmp
	python save_graph.py \
		-name-scope swa_model \
		-model-variables-prefix $(CHECKPOINT) \
		-model-config-json ../models/$(MODEL)/model.config.json

$(FROZEN_MODEL): $(MODEL_GRAPH)
	freeze_graph \
		--input_graph=$< \
		--input_checkpoint=$(CHECKPOINT) \
		--output_graph=$@ \
		--output_node_names=swa_model/policy_output,swa_model/value_output,swa_model/miscvalues_output,swa_model/ownership_output

$(OPTIMIZED_MODEL): $(FROZEN_MODEL)
	$(HOME)/OpenSources/tensorflow/bazel-bin/tensorflow/tools/graph_transforms/transform_graph \
		--in_graph=$< \
		--out_graph=$@ \
		--inputs="swa_model/bin_inputs,swa_model/global_inputs,swa_model/symmetries" \
		--outputs="swa_model/policy_output,swa_model/value_output,swa_model/miscvalues_output,swa_model/ownership_output" \
		--transforms='strip_unused_nodes fold_constants(ignore_errors=true) fold_batch_norms fold_old_batch_norms'  

$(SAVED_MODEL): $(OPTIMIZED_MODEL)
	python restore_saved_model.py $<

web_model: saved_model/saved_model.pb
	tensorflowjs_converter \
	    --input_format=tf_saved_model \
	    --saved_model_tags=serve \
	    saved_model \
	    web_model

clean:
	rm -rf saved_model web_model